# Document 07 — Research Notes and Alternatives

This file records the options considered before settling on the current direction. Keep it practical: add findings, tradeoffs, links, and test notes as the build evolves.

## Tuya local control options

### LocalTuya — selected starting point

LocalTuya is a common Home Assistant approach for controlling Tuya devices over the local network using device local keys. It keeps the normal control path local and avoids cloud round trips.

Why it fits:

- Works with existing Tuya WiFi bulbs.
- Integrates directly into Home Assistant entities.
- Lets ESPHome control a simple Home Assistant light group.

Risks:

- Local keys can be annoying to retrieve.
- Tuya datapoints vary by product and firmware.
- Bulb firmware updates may change behavior.

### tinytuya

A Python library for direct local Tuya control.

Why it was not selected first:

- Adds a separate script or service layer.
- More custom maintenance than using Home Assistant entities.
- Less convenient for ESPHome-driven interaction.

### Tuya-CloudCutter / custom firmware

A route for replacing Tuya firmware on some devices.

Why it was not selected first:

- Many newer Tuya devices are patched or use chips that are harder to convert.
- Risk is higher for bulbs already working with local control.
- The project goal is a reliable room controller, not bulb firmware replacement.

### Tuya Local integration

A possible fallback if LocalTuya is not reliable for the exact bulb model.

## Bedside input alternatives

### APDS-9960 gesture sensor — selected starting point

Why it fits:

- Touchless.
- Silent.
- Small and inexpensive.
- Supports both gestures and proximity.

Risks:

- Ambient light, mounting angle, and hand distance can affect reliability.
- Needs careful debounce to avoid accidental triggers.

### Capacitive touch pad

Good fallback if gestures are unreliable.

Pros:

- Very quiet.
- Simple hardware.
- Can be hidden under or behind a surface.

Cons:

- Still requires touching a surface.
- Can be sensitive to humidity and mounting materials.

### Accelerometer tap sensor

Could detect a tap on a nightstand.

Reason not selected:

- Easier to trigger accidentally from movement or vibration.

### mmWave presence radar

Better for room presence than explicit bedside commands.

Reason not selected:

- Too likely to react to normal sleeping movement.

## Door-side display alternatives

### ELECROW 1.28 in rotary display — selected starting point

Why it fits:

- Built-in round display, touch, ESP32-S3, and rotary input.
- Good match for a compact wall controller.
- Enough UI capability without requiring a separate app.

### Cheap Yellow Display / generic ESP32 touchscreen

Reason not selected:

- Larger and less polished for a bedroom door control.
- Resistive touch variants feel less premium.
- No native rotary interaction.

### M5Stack Dial / similar premium dial hardware

Possible alternative if the ELECROW board creates driver or enclosure issues.

## UI framework options

### ESPHome + LVGL — selected starting point

Why it fits:

- Keeps firmware declarative and easy to update OTA.
- Integrates with Home Assistant naturally.
- Good enough for a small, focused interface.

### SquareLine Studio / custom LVGL C++

Reason not selected first:

- More powerful, but adds build and maintenance overhead.
- Less convenient for quick ESPHome iteration.

### openHASP

Worth revisiting for touchscreen-first dashboards.

Reason not selected first:

- Rotary behavior and the specific ELECROW hardware path are simpler to keep inside ESPHome for this project.

## Inspiration

- Smart thermostat rotary UI patterns.
- SmartKnob-style physical-to-visual control mapping.
- Minimal Home Assistant dashboards.
- Small nightstand devices that stay useful without becoming visually loud.
