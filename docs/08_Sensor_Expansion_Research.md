# Document 08: Sensor Expansion Research & Integration Plan

**Author:** Manus AI (original), updated per owner approval
**Project:** VelaDial
**Date:** May 20, 2026 (original) — updated 2026-05-21

## Document status

This document was updated after PR #1 to align with the final architecture. Key alignments:

- The first-build architecture uses **two separate ESP32 nodes with separate I2C buses**, not one shared bus.
- **Door-side ESP32-S3** carries TSL2591 + SHT45.
- **Bedside ESP32-C6** carries APDS-9960 + VL53L4CD.
- The `0x29` address collision between TSL2591 and VL53L4CD is resolved **by architecture** (different nodes), not by XSHUT or a multiplexer.
- VL53L4CD ESPHome support is **unverified** and must be confirmed before production firmware. It is not assumed to require a custom component.

This document remains a research and alternatives reference. It does not duplicate `docs/02_TRD.md` or `docs/06_Implementation_Plan.md`.

---

This document details the research, specifications, and integration strategy for the expanded sensor array in the VelaDial project. The addition of three Adafruit sensors (TSL2591, VL53L4CD, SHT45) alongside the existing APDS-9960 enhances the system's environmental awareness and interaction reliability.

## 1. Sensor Array Specifications

The VelaDial system uses four I2C sensors distributed across two ESPHome nodes. Each node has its own independent I2C bus.

### 1.1 Door-side node (ESP32-S3, ELECROW rotary display)

| Sensor | Manufacturer Part | Primary Purpose | Key Specifications | I2C Address |
| :--- | :--- | :--- | :--- | :--- |
| **TSL2591** | Adafruit 1980 | High dynamic range ambient light sensing | 188 µLux to 88,000 Lux, 600M:1 range | `0x29` |
| **SHT45** | Adafruit 6174 | Precision temperature and humidity monitoring | ±0.1 °C, ±1 % RH accuracy | `0x44` |

### 1.2 Bedside node (ESP32-C6 Feather)

| Sensor | Manufacturer Part | Primary Purpose | Key Specifications | I2C Address |
| :--- | :--- | :--- | :--- | :--- |
| **APDS-9960** | Broadcom (Adafruit 3595) | Directional gesture detection (Left/Right/Up/Down) | 10–20 cm range, 2.4–3.6 V logic | `0x39` |
| **VL53L4CD** | Adafruit 5396 | Precise Time-of-Flight (ToF) distance measurement | 1 mm to 1300 mm range, 18° FoV | `0x29` (default) |

Each node uses STEMMA QT / Qwiic daisy-chaining *within* its own I2C bus. There is no cross-node sensor wiring.

## 2. I2C Architecture

Door-side and bedside nodes are separate ESP32 devices, each with its own independent I2C bus.

### 2.1 No first-build address conflict

TSL2591 and VL53L4CD both use the default I2C address `0x29`, but this is **not** a first-build conflict because the two sensors are on different ESP32 nodes. Each node's I2C bus only has one device at `0x29`.

- Door-side bus: TSL2591 (`0x29`) + SHT45 (`0x44`).
- Bedside bus: APDS-9960 (`0x39`) + VL53L4CD (`0x29`).

### 2.2 XSHUT / multiplexer is future fallback only

`XSHUT` address reassignment and a TCA9548A I2C multiplexer are **not required for the first build**. They are only future fallback options if the architecture ever changes to place both `0x29` sensors on the same I2C bus.

For reference, if both `0x29` sensors were ever placed on the same bus, the resolution path would be:

1. Wire the VL53L4CD `XSHUT` pin to a dedicated GPIO on the host ESP32.
2. On boot, pull `XSHUT` LOW to disable the VL53L4CD.
3. Initialize the TSL2591 at `0x29`.
4. Pull `XSHUT` HIGH to wake the VL53L4CD.
5. Send an I2C command to reassign the VL53L4CD to a new address (e.g., `0x30`).
6. Initialize the VL53L4CD at its new address.

If the breakout's `XSHUT` pin is not accessible (for example, if it is not exposed on the STEMMA QT daisy-chain side header), a TCA9548A I2C multiplexer would be required instead. **Neither path applies to the first build.**

## 3. Sensor Fusion & Interaction Strategy

The combination of these sensors allows for sensor fusion patterns where data from multiple sources is combined to create more reliable interactions. Fusion is a **useful design direction**, not a hard requirement for the first firmware build. Basic per-sensor behavior should be validated independently first, then fusion logic added on top.

### 3.1 Bedside Controller (ESP32-C6)

The bedside controller operates headlessly (no display) and relies on physical gestures.

- **Presence & Wake (VL53L4CD):** The ToF sensor monitors the space above the nightstand. When a hand enters a configured range (for example, 10–30 cm), it can wake or arm the APDS-9960 gesture detection. This is a fusion enhancement, not required for first firmware — APDS-9960 also has its own internal proximity detection.
- **Directional Control (APDS-9960):** Once active, the APDS-9960 detects directional swipes. A left swipe turns the `light.bedroom_group` OFF; a right swipe turns it ON.
- **Nightlight Mode (VL53L4CD):** If the ToF sensor detects a hand holding steady at a specific distance (for example, 5–10 cm) for more than ~1.5 seconds, it triggers a nightlight scene at low brightness and warm color temperature. The ToF sensor is more reliable for this static proximity detection than APDS-9960 proximity alone.

### 3.2 Door-Side Controller (ESP32-S3 + Rotary Display)

The door-side controller features the 1.28" LVGL display and uses environmental data to adapt its UI.

- **Adaptive Brightness (TSL2591):** The TSL2591 monitors ambient room light. In bright conditions the display backlight runs higher for readability; in dark conditions the backlight dims significantly to prevent light pollution. This affects the display only, not the room bulbs.
- **Environmental Context (SHT45):** The SHT45 provides precision temperature and humidity. It does not control lights and is treated as secondary diagnostics. Per the UI scope decision, environmental data does not appear as a required page on the main 3-page UI.

### 3.3 Fusion validation order

Before relying on fusion behavior:

1. Confirm APDS-9960 reliably reports clean left/right gestures on its own.
2. Confirm VL53L4CD reports stable distance readings on its own (including stable hold detection at 5–10 cm).
3. Only then layer the fusion: VL53L4CD distance gates / arms / refines APDS-9960 gesture handling, and VL53L4CD hand-hold triggers nightlight.

## 4. ESPHome Implementation Notes

| Sensor | ESPHome support status | Notes |
| :--- | :--- | :--- |
| **APDS-9960** | Native (`apds9960`) | Requires tuning of gesture sensitivity and proximity thresholds. Avoid extremely slow update intervals; gestures need fast polling. |
| **TSL2591** | Native (`tsl2591`) | Auto-gain recommended to span direct sunlight down to dark bedroom conditions. |
| **SHT45** | Native (`sht4x`) | The `sht4x` platform supports the SHT4x family including SHT45. |
| **VL53L4CD** | **Unverified** | Before writing production firmware, verify whether native ESPHome support, a maintained community component, or a custom component is available. If the support path is a blocker, evaluate a verified ESPHome-supported ToF fallback. **VL53L0X** has native ESPHome support and is the proven fallback. VL53L1X support has not been verified and should not be assumed without checking current official ESPHome documentation or a trusted component source. |

## 5. Power Budget Considerations

The ESP32-C6 is designed for low power, but running multiple active sensors continuously can draw notable current.

- The VL53L4CD and APDS-9960 both use active IR emitters.
- To reduce average current, the system can poll VL53L4CD at a lower frequency (for example, a few Hz) until proximity is detected, then increase APDS-9960 polling to catch fast gestures during the active window.
- The door-side node has no battery target (always USB powered), so its sensor polling can stay continuous.
- Actual mA draw should be measured during bring-up, not estimated, before any battery option is considered.

## 6. First-build decisions

These decisions are locked for the first build to keep scope tight and reviewable:

- **TSL2591** stays on the door-side node and controls adaptive display / backlight behavior only. It does not control room bulbs directly.
- **SHT45** stays on the door-side node and is secondary diagnostics only. It is exposed to Home Assistant but does not appear as a required element on the main 3-page UI.
- **APDS-9960** stays on the bedside node and is responsible for left/right directional gestures.
- **VL53L4CD** stays on the bedside node and is responsible for deliberate hand-hold nightlight detection (hand at ~5–10 cm for ~1.5 s).
- **Main UI** remains 3 pages only: Power, Brightness, Presets.
- **No required 4th Environment page** in the first UI build.
- **No HVAC or comfort automation** in the first build. Environmental data is diagnostic only.

## 7. Verification required before firmware

The following items must be validated on real hardware before any production firmware is written. None of these are firmware tasks themselves; they are prerequisites.

- **Verify VL53L4CD ESPHome support path.** Confirm native support, a maintained community component, or a custom component path. If blocked, switch to the verified VL53L0X fallback before writing bedside production YAML.
- **Verify ELECROW board revision and pinout.** Confirm the exact PCB revision via the board silkscreen and validate `hardware/elecrow_pinout.md` against it. Multiple ELECROW CrowPanel 1.28" revisions exist with subtly different pin maps.
- **Verify I2C scan on both nodes.** Confirm expected addresses appear on each node's bus (door-side: `0x29`, `0x44`; bedside: `0x39`, `0x29`). Confirm no unexpected addresses appear.
- **Verify TSL2591 readings in dark / dim / bright bedroom conditions.** Confirm auto-gain handles the full range without saturation or noise.
- **Verify SHT45 is not heat-biased by the display enclosure.** Mount SHT45 with airflow and thermal isolation from the display backlight; confirm readings are reasonable compared to a known-good reference.
- **Verify APDS-9960 gestures in the actual bedside mounting position.** Test against ambient IR (sunlight, lamps), expected hand approach angle, and typical false-trigger sources (covers, body warmth at 20–30 cm).

## References

[1] Adafruit Industries. "Adafruit TSL2591 High Dynamic Range Digital Light Sensor."
[2] Adafruit Industries. "Adafruit VL53L4CD Time of Flight Distance Sensor."
[3] Adafruit Industries. "Adafruit SHT45 Precision Temperature & Humidity Sensor."
[4] ESPHome Documentation. "APDS-9960 Gesture, Proximity, Light and RGB Sensor."
[5] ESPHome Documentation. "TSL2591 Light Sensor."
[6] ESPHome Documentation. "SHT4x Temperature/Humidity Sensor."
[7] ESPHome Documentation. "VL53L0X Time of Flight Distance Sensor."
[8] Home Assistant Community. Notes on APDS-9960 ESPHome integration and tuning.
