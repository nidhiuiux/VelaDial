# Document 03 — App Flow

This project has two user-facing surfaces: a round display by the door and a headless gesture controller near the bed.

## Door-side rotary display

### Default screen: Power

Purpose: quick room-level light control.

Flow:

1. User enters the room.
2. Display is dim or asleep.
3. Touch or knob movement wakes the display.
4. User taps the center power control.
5. Home Assistant toggles the bedroom light group.
6. Display updates the power state and optional LED ring.

### Brightness screen

Purpose: physical brightness control without a phone.

Flow:

1. User opens the brightness page.
2. User turns the rotary dial.
3. The on-screen arc changes in visible increments.
4. ESPHome sends a brightness update to Home Assistant.
5. Display keeps the new percentage visible briefly, then returns to low-brightness idle.

Recommended first-build behavior:

- 5% brightness increments.
- Clamp between 1% and 100% when on.
- Avoid sending commands on every tiny encoder bounce.

### Color preset screen

Purpose: choose simple room moods without a full color picker.

Initial presets:

- Warm white.
- Soft amber.
- Neutral white.
- Low nightlight.
- Optional RGB accent preset.

Flow:

1. User opens the preset screen.
2. User taps a preset.
3. Home Assistant applies the preset to the light group.
4. Display briefly confirms the selected preset.

### Idle and sleep behavior

- After about 60 seconds of inactivity, dim the display to 10%.
- Any touch or knob movement restores display brightness.
- Avoid bright white backgrounds or aggressive animations.

## Bedside gesture controller

The bedside node has no screen. It should behave like a quiet physical shortcut.

### Turn off

1. User waves left over the APDS-9960 sensor.
2. ESPHome receives a clean left gesture.
3. Home Assistant turns off the light group.

### Turn on

1. User waves right over the sensor.
2. ESPHome receives a clean right gesture.
3. Home Assistant turns on the light group.

### Nightlight

1. User holds a hand close to the sensor long enough to cross the proximity threshold.
2. Home Assistant turns the group on at low brightness, warm color.
3. A debounce/cooldown prevents repeated commands.

## Error handling

- Ignore unclear gestures instead of guessing.
- Do not flash lights on noisy sensor readings.
- If Home Assistant API is unavailable, fail quietly and recover automatically when the connection returns.
