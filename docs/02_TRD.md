# Document 02 — Technical Requirements

## System overview

VelaDial uses two ESPHome devices connected to Home Assistant over WiFi. Home Assistant owns the lighting entities and communicates with the Tuya bulbs locally. The ESPHome devices act as input surfaces and lightweight displays; they do not own long-term state.

## Hardware

### Door-side rotary display

- ELECROW CrowPanel 1.28 in rotary display.
- ESP32-S3 class board.
- GC9A01A round IPS display, 240 x 240.
- Rotary encoder and capacitive touch input.
- Optional WS2812 LED ring.

### Bedside gesture controller

- Adafruit ESP32-C6 Feather or equivalent ESP32-C6 board.
- APDS-9960 gesture/proximity sensor over I2C.
- Small bedside enclosure with the sensor exposed or behind a suitable window.

### Hub

- Home Assistant OS on Raspberry Pi 4/5 or another always-on local host.
- ESPHome add-on or ESPHome CLI for firmware build and OTA updates.
- HACS if LocalTuya is used.

## Software stack

- ESPHome for firmware configuration and device integration.
- Home Assistant as the automation/control hub.
- LocalTuya or another local Tuya LAN integration for RGB bulb control.
- LVGL inside ESPHome for the round display UI.

## Home Assistant entities

Expected entities should stay configurable, but the first build assumes:

- `light.bedroom_group` — group containing the five RGB bulbs.
- Individual Tuya bulb entities may be named differently in a real installation.

## Security and local configuration

Live network credentials, OTA credentials, API credentials, and Tuya local keys must stay outside the repository in the local ESPHome/Home Assistant configuration. The example firmware expects those values to be supplied by the local ESPHome secrets file.

## Functional requirements

- Left gesture turns the light group off.
- Right gesture turns the light group on.
- Proximity hold can trigger a warm 10% nightlight mode.
- Rotary encoder updates brightness in predictable increments.
- Touch UI supports power and preset selection.
- Display sleeps or dims after inactivity.

## Non-functional requirements

- Local LAN operation for normal use.
- No mechanical relay in the lighting control path.
- Fast response target: under 500 ms from user action to visible light change.
- Safe defaults after reboot.
- YAML should remain readable enough for manual debugging.

## Known implementation notes

- Pin mappings for the ELECROW display are tracked in `hardware/elecrow_pinout.md`.
- Display driver, touch transform, and encoder direction should be verified on real hardware.
- LocalTuya entity IDs and datapoints may vary by bulb model and firmware.
