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

## Must-have features

### Door-side controller

- ELECROW 1.28 in rotary display mounted near the door.
- Tap control for master on/off.
- Rotary control for brightness.
- Touch UI for simple color or temperature presets.
- Screen dims after inactivity to avoid lighting the room at night.

### Bedside controller

- ESP32-C6 with APDS-9960 gesture sensor.
- Left/right hand gestures for lights off/on.
- Optional proximity hold for low-brightness nightlight mode.
- No physical click required during normal use.

### Lighting control

- Controls a Home Assistant light group containing five Tuya RGB bulbs.
- Uses LocalTuya or an equivalent local LAN integration.
- Sends commands locally through Home Assistant.

## Nice-to-have features

- Ambient LED ring on the dial that reflects current lighting state.
- Warm nightlight preset with fixed brightness and color temperature.
- Auto-return to the power screen after a period of inactivity.
- Basic diagnostics page showing WiFi/API status.
- Haptic feedback later if hardware supports it.

## Out of scope for the first build

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

## Success criteria

- Gesture or knob action changes the light group in under 500 ms on the local network.
- The normal control path works when the internet is disconnected.
- No audible relay click, speaker response, or required voice command.
- The display is readable at arm’s length but not distracting in a dark room.
