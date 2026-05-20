# Project Context for AI Agents

Use this file to onboard an AI coding assistant onto VelaDial. It is written as a technical handoff, not as marketing copy.

## Project summary

**VelaDial** is a dual-node, silent, local-first smart lighting controller for a bedroom with multiple Tuya WiFi RGB bulbs.

The goal is to control the lights without voice commands, phone use, cloud latency, relay clicks, or bright screens at night.

The system has three parts:

1. **Door-side controller** — ELECROW 1.28 in ESP32-S3 rotary touch display running ESPHome and LVGL.
2. **Bedside controller** — ESP32-C6 with APDS-9960 gesture/proximity sensor running ESPHome.
3. **Home Assistant hub** — Raspberry Pi running Home Assistant with local Tuya bulb control.

## Current build goal

Write production-ready ESPHome YAML for:

1. The ELECROW 1.28 in rotary display with a full 3-page LVGL UI.
2. The bedside ESP32-C6 gesture controller.

The YAML should be readable, commented, configurable, and safe for public source control.

## Main design constraints

- Silent interaction only.
- Local-first control path.
- No internet dependency for normal light control.
- Target response below 500 ms on the local network.
- Dark UI with low light pollution.
- Large readable labels for a 240 x 240 round display.
- Keep credentials and local keys out of the repository.

## Hardware already selected

### Door-side controller

- ELECROW CrowPanel 1.28 in ESP32 rotary display.
- ESP32-S3R8 class board.
- 240 x 240 round IPS display.
- GC9A01A display over SPI.
- CST816D capacitive touch over I2C.
- Physical rotary encoder with press.
- 5-LED WS2812 ambient ring.
- 16 MB flash and 8 MB PSRAM on the purchased model.

### Bedside controller

- Adafruit ESP32-C6 Feather or equivalent ESP32-C6 board.
- APDS-9960 gesture/proximity sensor over I2C.
- Intended to be mounted in a small dark enclosure near the bed.

### Hub and lights

- Home Assistant runs on a Raspberry Pi.
- The target room has five Tuya WiFi RGB bulbs.
- Bulbs should be grouped in Home Assistant as one controllable light group.
- LocalTuya or an equivalent local Tuya LAN integration is the preferred control path.

## Decisions already made

- Door-side controller: ELECROW 1.28 in rotary display.
- Bedside controller: ESP32-C6 plus APDS-9960.
- Firmware: ESPHome for both controllers.
- UI framework: ESPHome LVGL.
- Hub: Home Assistant.
- Bulb control: local Tuya LAN control through Home Assistant.
- Communication: ESPHome Native API to Home Assistant, then Home Assistant to the light group.

## ELECROW rotary display pinout

### Display — GC9A01A over SPI

| Function | GPIO |
| --- | ---: |
| SCLK | 10 |
| MOSI | 11 |
| MISO | not used |
| DC | 3 |
| CS | 9 |
| RST | 14 |
| Backlight | 46 |

### Touch — CST816D over I2C

| Function | GPIO |
| --- | ---: |
| SDA | 6 |
| SCL | 7 |
| INT | 5 |
| RST | 13 |

### Rotary encoder

| Function | GPIO |
| --- | ---: |
| Encoder A | 45 |
| Encoder B | 42 |
| Switch press | 41 |

### RGB LED ring — WS2812

| Function | GPIO / Value |
| --- | ---: |
| Data | 48 |
| LED count | 5 |

## ESPHome notes for ELECROW display

- Board: `esp32-s3-devkitc-1`.
- Framework: ESP-IDF.
- Display model: `GC9A01A`.
- Touch platform: `cst816`, address `0x15`.
- `invert_colors: true` may be required.
- PSRAM and flash settings should match the board revision.
- Touch transform and encoder direction must be verified on real hardware.

## Door-side LVGL UI requirements

The display should use a 3-page swipeable LVGL interface.

### Page 1 — Power

- Large central power control.
- Tap to toggle the bedroom light group.
- Ambient LED ring glows amber when lights are on.
- Clear on/off state text.

### Page 2 — Brightness

- Circular arc around the edge of the round display.
- Large center percentage label, such as `75%`.
- Rotary knob changes brightness in 5% steps.
- Commands should be debounced enough to avoid flooding Home Assistant.

### Page 3 — Color / temperature

- Simple color-temperature or preset page.
- 4 to 6 large preset buttons.
- Suggested presets: red, blue, purple, warm white, neutral white, nightlight.

## UI design rules

- Background: pure black, `#000000`.
- Primary accent: warm amber, `#FFB300`.
- Text: white, `#FFFFFF`.
- Large primary font size, roughly 48 px or larger where practical.
- Flat design, no heavy shadows.
- Screen dims to about 10% after 60 seconds of inactivity.
- Touch or knob movement wakes the screen.
- Waking the screen should not accidentally toggle the lights.

## Bedside gesture requirements

The bedside device is headless and touchless.

Expected interactions:

- Wave left: turn all bedroom lights off.
- Wave right: turn all bedroom lights on.
- Hold hand over sensor with proximity above threshold: nightlight mode at about 10% brightness, warm white.
- Use a cooldown after each action to reduce false triggers.

## Recommended Home Assistant entity model

Keep this configurable in YAML substitutions:

- `light.bedroom_group` — group of the five bedroom bulbs.

Do not hardcode private device keys, tokens, passwords, or local environment values.

## What another AI should do next

1. Read `README.md`, this file, and all files in `docs/`.
2. Treat the pinout in `hardware/elecrow_pinout.md` as the local working reference.
3. Upgrade `esphome/door_side_rotary.yaml` from bring-up firmware to a full LVGL UI.
4. Keep `esphome/bedside_gesture.yaml` simple, reliable, and debounced.
5. Do not introduce a cloud-only dependency into the main control path.
6. Clearly mark any hardware assumption that still needs real-device validation.

## Useful references

- ELECROW CrowPanel 1.28 in HMI ESP32 rotary display documentation.
- ESPHome LVGL documentation.
- ESPHome APDS9960 documentation.
- Home Assistant LocalTuya documentation and community notes.
- SmartKnob-style rotary interaction patterns as inspiration, not as code to copy.
