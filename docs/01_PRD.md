# Document 01 — Product Requirements

## Product name

**VelaDial**

## One-line description

A quiet, local-first lighting controller for bedroom RGB bulbs, with a door-side rotary display and a bedside gesture sensor.

## Problem

Bedroom lighting should be easy to control without using a phone, voice assistant, or noisy switch. The controller needs to work from two places: near the door when entering the room and near the bed when turning lights off or using a low nightlight mode.

The system should feel instant, quiet, and private. Internet access should not be required for normal operation.

## Target user

A smart-home builder who wants a custom physical interface for local RGB lighting control, values low-noise operation, and is comfortable using Home Assistant, ESPHome, and small ESP32-based hardware.

## Hardware inventory

### Door-side controller

- ELECROW ESP32-S3 rotary display (1.28 in, 240 x 240, GC9A01A).
- TSL2591 ambient light sensor (I2C `0x29`).
- SHT45 temperature/humidity sensor (I2C `0x44`).

### Bedside controller

- ESP32-C6 (Adafruit Feather or equivalent).
- APDS-9960 gesture sensor (I2C `0x39`).
- VL53L4CD time-of-flight distance sensor (I2C `0x29`).

### Hub

- Home Assistant OS on Raspberry Pi 4/5 or equivalent always-on local host.

## Must-have features

### Door-side controller

- ELECROW 1.28 in rotary display mounted near the door.
- Tap control for master on/off.
- Rotary control for brightness.
- Touch UI for simple color or temperature presets.
- Screen dims after inactivity to avoid lighting the room at night.
- Display must support adaptive brightness using ambient light data from TSL2591.
- Display brightness must be safe for bedroom use at night.

### Bedside controller

- ESP32-C6 with APDS-9960 gesture sensor.
- Left/right hand gestures for lights off/on.
- Reliable hand-hold nightlight behavior using VL53L4CD distance sensing.
- APDS-9960 remains responsible for directional left/right gestures.
- Sensor behavior must not cause accidental full-brightness light changes.
- No physical click required during normal use.

### Lighting control

- Controls a Home Assistant light group containing five Tuya RGB bulbs.
- Uses LocalTuya or an equivalent local LAN integration.
- Sends commands locally through Home Assistant.

### Main door-side UI pages

The main door-side UI remains focused on three pages:

1. Power
2. Brightness
3. Presets

## Nice-to-have features

- Ambient LED ring on the dial that reflects current lighting state.
- Warm nightlight preset with fixed brightness and color temperature.
- Auto-return to the power screen after a period of inactivity.
- Basic diagnostics page showing WiFi/API status.
- Haptic feedback later if hardware supports it.
- Temperature and humidity display from SHT45.
- Environmental diagnostics view (secondary, not a main page).
- Ambient-aware LED ring intensity (adapts to room brightness).
- Comfort-aware Home Assistant automations.
- SHT45 data shown in Home Assistant dashboard.

## Out of scope for the first build

- No 4th required environment page in the first UI build.
- No cloud dependency for sensor behavior.
- No automatic comfort/HVAC control in the first build.
- No production firmware changes in this documentation step.
- No second SHT45 sensor for bedside in the first build.
- No I2C multiplexer required for the first-build architecture.
- Cloud-only Tuya control.
- Voice assistant dependency for the main workflow.
- Complex animations, video playback, or heavy graphics on the display.
- Mobile app development.
- Full scene editor inside the ESPHome UI.

## User stories

- As someone entering the bedroom, I want to tap the door-side display so the room lights turn on immediately.
- As someone adjusting the room mood, I want to rotate a physical dial so brightness changes smoothly without opening an app.
- As someone already in bed, I want to wave my hand once to turn lights off silently.
- As someone waking up at night, I want a low warm nightlight without turning the room fully on.
- As a user entering the room in daylight, I want the display to be bright enough to read.
- As a user entering the room at night, I want the display to stay dim and not disturb sleep.
- As a user in bed, I want to hold my hand near the bedside sensor to trigger a low warm nightlight.
- As a user, I want directional gestures to remain reliable and not accidentally trigger from vague proximity.
- As a user, I may want to check room temperature/humidity later, but this should not interfere with lighting control.

## Success criteria

- Gesture or knob action changes the light group in under 500 ms on the local network.
- The normal control path works when the internet is disconnected.
- No audible relay click, speaker response, or required voice command.
- The display is readable at arm's length but not distracting in a dark room.
- Display backlight adapts smoothly to room brightness without flicker.
- Night mode display brightness does not create distracting light pollution.
- VL53L4CD hand-hold nightlight requires deliberate hold behavior, not a quick pass-by.
- Directional gesture control still works after adding VL53L4CD.
- Main lighting actions remain local-first and fast.
- Environmental data is secondary and does not clutter the main UI.
