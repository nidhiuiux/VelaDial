# Document 05 — Entity Model and Local Architecture

This project does not have a conventional web-app backend. Home Assistant is the local hub, and ESPHome devices communicate with it through the native API.

## Architecture

```text
Door-side ESPHome dial
Bedside ESPHome sensor
        ↓ WiFi LAN
Home Assistant
        ↓ local LAN integration
Tuya RGB bulbs
```

The ESPHome nodes are mostly stateless. They send user intent to Home Assistant and optionally display the current state.

## Home Assistant lighting model

### Light group

Recommended group:

- `light.bedroom_group`

Members:

- Five Tuya RGB bulb entities.
- Actual entity IDs may differ after LocalTuya setup.

Reason for grouping:

- One command controls all bulbs.
- ESPHome YAML stays simple.
- Future bulbs can be added in Home Assistant without changing firmware.

## ESPHome entities

### Door-side dial

Expected entities:

- Rotary position sensor.
- Rotary button sensor.
- Touch wake event.
- Backlight control.
- Optional LED ring light entity.

### Bedside gesture node

Expected entities:

- Left gesture sensor.
- Right gesture sensor.
- Proximity sensor.

Exact generated names depend on ESPHome naming and friendly-name settings.

## Actions

Initial Home Assistant actions:

- Turn the group off for a left gesture.
- Turn the group on for a right gesture.
- Turn the group on at low brightness for nightlight.
- Set brightness from the rotary control.

## Sensitive data

Do not commit live WiFi credentials, API credentials, Home Assistant tokens, Tuya local keys, or generated ESPHome build output. Keep private values in the local ESPHome/Home Assistant configuration.

## Configuration values to keep easy to change

- Light group entity ID.
- Gesture threshold and cooldown.
- Screen dim timeout.
- Brightness step size.
- Color presets.
- Rotary encoder direction.
- Touch transform settings.
