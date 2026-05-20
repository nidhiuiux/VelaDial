# Sensor Expansion Change Plan

**Author:** Manus AI  
**Date:** May 20, 2026  
**Status:** DRAFT — Awaiting owner review

This document describes the controlled set of changes required to integrate the three new Adafruit sensors (TSL2591, VL53L4CD, SHT45) into the VelaDial project documentation and firmware architecture. No changes will be made until the owner explicitly approves each step.

---

## New Hardware Being Added

| Part | Adafruit # | Mouser # | I2C Address | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| TSL2591 High Dynamic Range Light Sensor | 1980 | 485-1980 | `0x29` (fixed) | Ambient light sensing for adaptive display brightness and night/day UI adaptation |
| VL53L4CD Time-of-Flight Distance Sensor | 5396 | 485-5396 | `0x29` (default, programmable) | Reliable hand-near/hold detection for bedside nightlight mode and display wake |
| SHT45 Precision Temp & Humidity Sensor | 6174 | 485-6174 | `0x44` | Bedroom comfort diagnostics, optional environmental display page |

---

## Documents That Need Changes

### 01_PRD.md
- Add the three new sensors to the hardware inventory section.
- Add "adaptive display brightness" as a product feature.
- Add "environmental display page" as an optional/future feature.
- Add "reliable nightlight hold gesture via ToF" to bedside interaction requirements.

### 02_TRD.md
- Add TSL2591, VL53L4CD, SHT45 to the hardware components table.
- Document the I2C address conflict (TSL2591 vs VL53L4CD both at `0x29`) and the XSHUT-based resolution strategy.
- Add VL53L4CD custom component requirement (not natively supported in ESPHome).
- Update the I2C bus diagram to show all 4 sensors daisy-chained via STEMMA QT.
- Add power budget notes for the expanded sensor array.

### 03_App_Flow.md
- Add a new interaction: "VL53L4CD detects hand → wakes APDS-9960 → APDS reads gesture."
- Add the nightlight hold interaction: "Hand steady at 5-10cm for 1.5s → nightlight scene."
- Add adaptive brightness flow: "TSL2591 lux reading → backlight PWM adjustment."
- Add optional environment page navigation to the door-side flow.

### 04_UI_UX_Design_Brief.md
- Add an optional 4th page concept: "Environment" showing temperature and humidity.
- Document the adaptive backlight behavior (bright in day, dim at night).
- Note that the TSL2591 data may also influence the amber accent intensity.

### 05_Backend_Schema.md
- Add new ESPHome sensor entities: `sensor.bedroom_ambient_light`, `sensor.bedroom_temperature`, `sensor.bedroom_humidity`, `sensor.bedside_distance`.
- Add the VL53L4CD custom component dependency.
- Update the I2C bus architecture section.

### 06_Implementation_Plan.md
- Add a new phase (or sub-phase) for sensor hardware assembly and I2C address conflict resolution.
- Add a testing step for verifying all 4 sensors communicate on the shared bus.
- Add the VL53L4CD custom component integration step.
- Update the bedside firmware phase to include sensor fusion logic.

### 07_Research_Alternatives.md
- Add a section comparing the VL53L4CD to other ToF sensors (VL53L0X, VL53L1X, VL6180X).
- Add a section comparing the TSL2591 to other light sensors (BH1750, VEML7700, TSL2561).
- Add a section comparing the SHT45 to other temp/humidity sensors (SHT40, BME280, AHT20).

---

## Documents That Do NOT Need Changes

| File | Reason |
| :--- | :--- |
| `README.md` | Will be updated only after all doc changes are approved and merged. |
| `AGENTS.md` | Agent workflow is unchanged. |
| `PROJECT_CONTEXT_FOR_AI.md` | Will be regenerated after all changes are finalized. |
| `esphome/door_side_rotary.yaml` | Production firmware — not touched in this step. |
| `esphome/bedside_gesture.yaml` | Production firmware — not touched in this step. |
| `hardware/elecrow_pinout.md` | Unrelated to the new sensors (covers the display only). |

---

## Files That Will Be Created (New)

| File | Purpose |
| :--- | :--- |
| `docs/08_Sensor_Expansion_Research.md` | Already created on this branch. Contains research findings, specs, and sensor fusion strategy. |
| `hardware/sensor-wiring.md` | New file documenting the I2C bus wiring, STEMMA QT daisy-chain order, and XSHUT pin connection. |

---

## Proposed Execution Order

| Step | Action | Scope |
| :--- | :--- | :--- |
| 1 | Create this change plan (this file) | `docs/sensor-expansion-change-plan.md` |
| 2 | Owner reviews and approves the plan | — |
| 3 | Update `02_TRD.md` (hardware table, I2C conflict, power budget) | Controlled edit |
| 4 | Update `01_PRD.md` (hardware inventory, new features) | Controlled edit |
| 5 | Update `05_Backend_Schema.md` (new entities, bus architecture) | Controlled edit |
| 6 | Update `03_App_Flow.md` (new interactions, sensor fusion flow) | Controlled edit |
| 7 | Update `04_UI_UX_Design_Brief.md` (environment page, adaptive brightness) | Controlled edit |
| 8 | Update `06_Implementation_Plan.md` (new phases) | Controlled edit |
| 9 | Update `07_Research_Alternatives.md` (sensor comparisons) | Controlled edit |
| 10 | Create `hardware/sensor-wiring.md` | New file |
| 11 | Owner reviews all changes | — |
| 12 | Create PR for owner to merge (only when explicitly approved) | PR |

---

## Open Questions for Owner

1. Should the TSL2591 be mounted on the **door-side** controller (near the display it controls) or on the **bedside** controller (closer to the sleeping area)?
2. Should the SHT45 be on the door-side or bedside? (Or both?)
3. For the VL53L4CD XSHUT pin: are you comfortable soldering one extra wire from the breakout to a GPIO, or would you prefer to buy a TCA9548A I2C multiplexer instead?
4. Should the "Environment" page be a 4th swipeable page on the rotary display, or accessible only via a long-press/special gesture?
