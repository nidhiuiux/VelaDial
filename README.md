# VelaDial

A quiet, local-first bedroom lighting controller built with Home Assistant, ESPHome, a round rotary display, and a bedside gesture sensor.

VelaDial controls a group of Tuya RGB bulbs without relying on voice commands, cloud latency, relay clicks, or a phone screen at night.

## Hardware concept

- Door-side ELECROW 1.28 in rotary touch display for power, brightness, and color presets.
- Bedside ESP32-C6 with APDS-9960 gesture sensor for silent hand-wave control.
- Home Assistant as the local hub.
- LocalTuya or an equivalent local LAN integration for the existing Tuya bulbs.

### Sensor expansion

VelaDial includes additional sensors for adaptive behavior and improved reliability:

| Node | Sensor | I2C Address | Purpose |
| --- | --- | --- | --- |
| Door-side ESP32-S3 | TSL2591 | `0x29` | Adaptive display brightness based on ambient light |
| Door-side ESP32-S3 | SHT45 | `0x44` | Secondary temperature/humidity diagnostics |
| Bedside ESP32-C6 | APDS-9960 | `0x39` | Directional gestures (left/right) for light on/off |
| Bedside ESP32-C6 | VL53L4CD | `0x29` | Deliberate hand-hold detection for nightlight mode |

The door-side and bedside nodes use separate I2C buses. TSL2591 and VL53L4CD both use `0x29`, but this is not a conflict because they are on different ESP32 devices. No I2C multiplexer is required for the first build.

### Key sensor roles

- TSL2591 controls adaptive display brightness only. It does not control room bulbs directly.
- SHT45 provides secondary diagnostics only. It is not a required main page element.
- VL53L4CD enables deliberate hand-hold nightlight behavior (stable hand at 5–10 cm for ~1.5 s).
- APDS-9960 remains the directional gesture sensor for light on/off.

## Main UI

The door-side display has exactly 3 primary pages:

1. Power
2. Brightness
3. Presets

Temperature/humidity data is secondary and does not appear on the main pages.

## Repository layout

```text
.
├── PROJECT_CONTEXT_FOR_AI.md
├── AGENTS.md
├── docs/
├── esphome/
└── hardware/
```

## Start here

For a coding assistant, paste or open `PROJECT_CONTEXT_FOR_AI.md` first. It contains the hardware selection, pinout, sensor architecture, UI requirements, and next implementation target.

For human review, read in this order:

1. `PROJECT_CONTEXT_FOR_AI.md`
2. `docs/01_PRD.md`
3. `docs/02_TRD.md`
4. `docs/03_App_Flow.md`
5. `docs/04_UI_UX_Design_Brief.md`
6. `docs/05_Backend_Schema.md`
7. `docs/06_Implementation_Plan.md`
8. `docs/07_Research_Alternatives.md`
9. `esphome/`
10. `hardware/`

## Current firmware status

The ESPHome files are starter bring-up configurations, not final production firmware.

- `esphome/door_side_rotary.yaml` brings up the ELECROW display, touch, encoder, backlight, and basic LVGL page.
- `esphome/bedside_gesture.yaml` brings up the ESP32-C6 + APDS-9960 gesture controller.

The next major work items are validation-first, then bring-up firmware. Production YAML is not the next step:

1. Validate the ELECROW board revision and pinout against the actual hardware before any production firmware.
2. Verify I2C scans on both nodes (door-side: `0x29`, `0x44`; bedside: `0x39`, `0x29`).
3. Verify the VL53L4CD ESPHome support path before writing hold-nightlight firmware. If blocked, surface the decision to the owner before any fallback.
4. Start with bring-up firmware, not full production YAML.
5. Door-side bring-up: display, touch, encoder, backlight, TSL2591, SHT45.
6. Bedside bring-up: APDS-9960 standalone first; VL53L4CD support verification second. **Do not implement sensor fusion in v1.**

v1 bedside behavior is APDS-9960 standalone left/right gestures plus VL53L4CD standalone hand-hold nightlight (only if VL53L4CD support is verified). VL53L4CD/APDS-9960 sensor fusion is v2 / future only — not a first-build requirement.

## Design principles

- Silent by default.
- Local-first control path.
- Low light pollution for bedroom use.
- Small, reviewable firmware and documentation changes.
- Useful for AI-assisted development, but still human-reviewed.

## Private configuration

Keep live WiFi credentials, API credentials, Tuya local keys, Home Assistant tokens, and generated ESPHome build output out of the repository. Store private values only in your local ESPHome/Home Assistant configuration.
