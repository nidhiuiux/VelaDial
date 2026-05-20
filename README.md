# VelaDial

A quiet, local-first bedroom lighting controller built with Home Assistant, ESPHome, a round rotary display, and a bedside gesture sensor.

VelaDial controls a group of Tuya RGB bulbs without relying on voice commands, cloud latency, relay clicks, or a phone screen at night.

## Hardware concept

- Door-side ELECROW 1.28 in rotary touch display for power, brightness, and color presets.
- Bedside ESP32-C6 with APDS-9960 gesture sensor for silent hand-wave control.
- Home Assistant as the local hub.
- LocalTuya or an equivalent local LAN integration for the existing Tuya bulbs.

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

For a coding assistant, paste or open `PROJECT_CONTEXT_FOR_AI.md` first. It contains the hardware selection, pinout, UI requirements, and next implementation target.

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

The next major work item is upgrading the door-side YAML into the full 3-page LVGL UI described in `PROJECT_CONTEXT_FOR_AI.md`.

## Design principles

- Silent by default.
- Local-first control path.
- Low light pollution for bedroom use.
- Small, reviewable firmware and documentation changes.
- Useful for AI-assisted development, but still human-reviewed.

## Private configuration

Keep live WiFi credentials, API credentials, Tuya local keys, Home Assistant tokens, and generated ESPHome build output out of the repository. Store private values only in your local ESPHome/Home Assistant configuration.
