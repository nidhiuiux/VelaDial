# Document 03 — App Flow

This project has two user-facing surfaces: a round display by the door and a headless gesture controller near the bed.

## High-level flow diagram

```text
Door-side:
  User input / TSL2591 lux
  → ESPHome door-side logic
  → LVGL display + backlight behavior
  → Home Assistant light commands when needed

Bedside:
  VL53L4CD hand distance
  → Arm APDS-9960 or detect hold
  → Gesture / nightlight decision
  → Home Assistant light command
  → Bedroom light group
```

## Door-side rotary display

The main door-side UI has exactly 3 primary pages:

1. Power
2. Brightness
3. Presets

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

### Wake behavior clarification

- Touch and knob rotation while display is asleep should wake the screen first.
- Whether physical knob press toggles power while asleep remains a design decision to test.
- Avoid accidental room-light changes from bumps.
- Bedroom safety is more important than shortcut speed.

### Adaptive display brightness flow

TSL2591 measures ambient lux near the door-side display. ESPHome maps the lux reading to a display backlight target. This runs locally on the door-side ESP32-S3, without Home Assistant round-trip.

Flow:

1. TSL2591 reads ambient lux continuously.
2. ESPHome maps lux to a backlight PWM target using a configurable curve.
3. Bright room = display backlight increases for readability.
4. Low-light/night = display stays dim to avoid disturbing sleep.
5. Backlight changes are smoothed to prevent flicker.
6. User interaction can wake the screen, but ambient rules still cap night brightness.
7. TSL2591 is for display/UI adaptation, not room bulb control.

## Bedside gesture controller

The bedside node has no screen. It should behave like a quiet physical shortcut.

### Bedside sensor-fusion gesture flow

VL53L4CD monitors distance near the bedside sensor. It improves presence/distance reliability. APDS-9960 remains responsible for directional gestures.

Flow:

1. VL53L4CD monitors distance near the bedside sensor.
2. Hand enters configured range = hand_near / hand_present.
3. Hand-near arms or wakes APDS-9960 gesture detection.
4. APDS-9960 reads directional gesture.
5. Valid left gesture turns bedroom group off.
6. Valid right gesture turns bedroom group on.
7. Cooldown prevents repeated triggers.
8. If no valid gesture is detected after timeout, return to idle.

### ToF-based nightlight hold flow

User holds hand near bedside sensor to trigger a low warm nightlight without directional gestures.

Flow:

1. User holds hand near bedside sensor.
2. VL53L4CD detects stable distance around 5–10 cm.
3. ESPHome starts hold timer.
4. If hand remains steady for about 1.5 seconds, bedside_nightlight_hold becomes active.
5. Home Assistant receives nightlight command.
6. Bedroom group turns on warm color at about 10% brightness.
7. Cooldown prevents repeated nightlight triggers.
8. If hand moves away before hold time, no action happens.
9. Quick pass-by motion must not trigger nightlight.

### Turn off (gesture)

1. Hand enters range, VL53L4CD arms APDS-9960.
2. User waves left over the APDS-9960 sensor.
3. ESPHome receives a clean left gesture.
4. Home Assistant turns off the light group.
5. Cooldown blocks further gestures for 1 second.

### Turn on (gesture)

1. Hand enters range, VL53L4CD arms APDS-9960.
2. User waves right over the sensor.
3. ESPHome receives a clean right gesture.
4. Home Assistant turns on the light group.
5. Cooldown blocks further gestures for 1 second.

## Environmental diagnostics flow

SHT45 measures room temperature and humidity on the door-side node.

- Values are exposed to Home Assistant.
- Values may appear later in a diagnostics/info view.
- Values may appear in Home Assistant dashboard.
- Environmental data does not affect the main Power/Brightness/Presets flow in first build.
- No HVAC or comfort automation is required in first build.

## Sensor failure fallback behavior

- If TSL2591 is unavailable, use fixed display dim/sleep behavior.
- If SHT45 is unavailable, hide/ignore environmental diagnostics.
- If VL53L4CD is unavailable, bedside may fall back to APDS-9960 gesture-only behavior.
- If APDS-9960 is unavailable, distance-based nightlight hold may still work if VL53L4CD works.
- Sensor failure must not cause full-brightness light changes.
- Fail quietly and recover automatically when sensor returns.

## Error handling

- Ignore unclear gestures instead of guessing.
- Do not flash lights on noisy sensor readings.
- If Home Assistant API is unavailable, fail quietly and recover automatically when the connection returns.
