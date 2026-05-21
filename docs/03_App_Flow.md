# Document 03 — App Flow

This project has two user-facing surfaces: a round display by the door and a headless gesture controller near the bed.

## High-level flow diagram

```text
Door-side:
  User input / TSL2591 lux
  → ESPHome door-side logic
  → LVGL display + backlight behavior
  → Home Assistant light commands when needed

Bedside (v1):
  APDS-9960 directional gesture        →  Home Assistant light command  →  Bedroom light group
  VL53L4CD hand-hold (≈5–10 cm, ≈1.5 s) →  Home Assistant nightlight cmd →  Bedroom light group

Bedside sensor fusion (v2 / future):
  VL53L4CD distance arms or refines APDS-9960 gesture detection (not required for v1)
```

## Door-side rotary display

The main door-side UI has exactly 3 primary pages:

1. Power
2. Brightness
3. Presets

The main UI stays exactly 3 pages — Power, Brightness, Presets. There is **no** 4th Environment page in the first build.

### Page navigation

- Horizontal swipe navigates between pages.
- Left swipe moves to the next page (Power → Brightness → Presets → Power).
- Right swipe moves to the previous page.
- A small 3-dot page indicator appears near the bottom of the round screen.
- The active page's dot is amber; inactive dots are white at low contrast.
- No 4th Environment page on the main UI.

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

First-build presets (locked, exactly 4):

1. Warm White
2. Soft Amber
3. Neutral White
4. Low Nightlight

Layout and behavior:

- Display as 4 large touch targets in a 2×2 grid on the round screen.
- Tap a preset to apply it to the bedroom light group.
- The currently active preset is highlighted with an amber border.
- Display briefly confirms the selected preset.

Out of first-build scope:

- Optional RGB accent preset is deferred to a future / v2 build.
- No dense color picker, color wheel, or per-bulb color editor in first build.

### Idle and sleep behavior

- After about 60 seconds of inactivity, dim the display to 10%.
- Any touch or knob movement restores display brightness.
- Avoid bright white backgrounds or aggressive animations.

### Wake behavior clarification

This behavior is locked for the first build:

- **Touch while asleep:** wake only.
- **Knob rotation while asleep:** wake only.
- **Knob press while asleep:** wake only.
- A second deliberate action while the display is awake is what toggles or sends a command.
- Avoid accidental room-light changes from bumps.
- Bedroom safety is more important than shortcut speed.

### Knob press behavior per page

These behaviors apply only when the display is already awake.

- **Power page:** knob press toggles the bedroom light group (same as tapping the center).
- **Brightness page:** knob press returns to the Power page.
- **Presets page:** knob press applies the currently highlighted preset.
- **While asleep:** knob press wakes the display only; it does not toggle, apply, or navigate.

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

### Bedside gesture flow (v1)

In the first build, APDS-9960 handles directional gestures on its own. VL53L4CD does **not** arm or gate the gesture sensor in v1.

- **Left gesture** (wave left over the APDS-9960 sensor): turns the bedroom light group off.
- **Right gesture** (wave right over the sensor): turns the bedroom light group on.
- A cooldown (about 1 s) blocks further gestures after a successful gesture to prevent repeat-fire.
- Unclear, ambiguous, or noisy gesture reads are ignored; APDS-9960 misses are expected in real-world use.
- Mounting and ambient lighting (sunlight, IR sources) affect reliability; see `docs/02_TRD.md` sensor timing notes.

### ToF nightlight hold flow (v1)

VL53L4CD handles deliberate hand-hold detection on its own in the first build. It does **not** arm APDS-9960.

- **Trigger:** hand stable in the 5–10 cm range above the bedside sensor.
- **Hold duration:** continuous in-range for about 1.5 s.
- **Hand-near debounce:** require the hand to be in range for at least about 200 ms before the 1.5 s hold timer starts. Prevents quick pass-by motion from starting the timer.
- **Cooldown:** about 5 s after a successful trigger before another nightlight trigger is possible.
- **Action on trigger:** Home Assistant turns the bedroom group on at a low warm nightlight (about 10 % brightness, warm color).
- **Quick pass-by motion must not trigger.**
- **Hand moves away before hold completes:** no action; the hold timer resets.

**Sensor support note for VL53L4CD.** VL53L4CD ESPHome support is unverified and must be confirmed before production firmware. If support becomes blocked during validation, evaluate **VL53L0X** as a verified ESPHome-supported ToF fallback only after explicit owner approval. VL53L0X has a higher minimum range than VL53L4CD, so the 5–10 cm hold threshold would need to be re-tuned. VL53L0X is **not** the selected default — VL53L4CD remains the planned sensor.

### Bedside sensor fusion (v2 / future)

Sensor fusion is **not** part of the first build. It is described here only as a future enhancement path.

- VL53L4CD distance reading could be used to arm, wake, or refine APDS-9960 gesture detection — for example, only accepting APDS-9960 gesture events while VL53L4CD reports a hand within an arming range.
- Fusion should only be added after both APDS-9960 standalone gestures (v1) and VL53L4CD standalone hold (v1) are independently verified to work reliably in the actual bedside mounting.
- See `docs/08_Sensor_Expansion_Research.md` for the broader fusion design discussion.

### Bedside status LED behavior (optional, v1)

An optional small status LED on the bedside controller provides silent local feedback. This is optional first-build hardware, not a required addition.

- **1 blink:** input detected (gesture or button registered locally by the ESP32).
- **2 blinks:** nightlight hold triggered.
- The status LED does **not** confirm bulb state. Bulb state is shown by the bulbs themselves. The LED only confirms that the ESP32 saw the input.
- v1 does **not** use a steady or pulsing offline-status LED. A constantly lit LED on a bedside device causes light pollution in a dark bedroom, which contradicts the project's bedroom-safe principle.
- Offline / unavailable status belongs primarily on the door-side display, not on a constantly lit bedside LED.

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

## Connectivity and outage behavior

VelaDial is local-first. Normal control runs over the local LAN through Home Assistant and LocalTuya. The following describes user-visible behavior when parts of that path are unavailable. Infrastructure setup recommendations (router, WiFi, UPS, static DHCP, etc.) live in `docs/02_TRD.md`, not here.

### Internet / ISP outage

- If the local WiFi / LAN, Raspberry Pi, Home Assistant, and LocalTuya are all healthy, normal VelaDial control should continue without change.
- The user should not see a special "internet offline" state for normal local control.
- No cloud dependency is added; internet outage by itself is not a user-visible failure mode.

### WiFi / router / AP / LAN instability

- If ESP32 nodes, the Raspberry Pi, or the Tuya bulbs disconnect from the local LAN, light commands may not reach the bulbs.
- The door-side display should not pretend a command succeeded if Home Assistant does not confirm the new state.
- The bedside controller may provide local status LED feedback that the gesture or button was detected, but the bulb action still depends on the LAN / Home Assistant path.
- The system should recover quietly when the LAN returns. No loud error chimes, no flashing animations, no startup popups.

### Home Assistant / Raspberry Pi unavailable

- Door-side controls (touch, knob rotation, knob press) may still wake and navigate the display locally, but light commands may fail.
- Bedside gestures may still be detected locally, but the corresponding bulb commands may fail.
- The UI should show a simple offline / unavailable state when it can — for example, dim the power indicator, gray out brightness percentage, or show a brief "unavailable" label — rather than display false-success state.
- Avoid repeated command spam while Home Assistant is unavailable. Retry with backoff, not on every input.
- Recover automatically when Home Assistant returns. Do not require the user to do anything.

### Door-side behavior notes (when state is uncertain)

- User input while the display is asleep wakes the display first; the second deliberate action toggles power or sends a command (see "Wake behavior clarification" above).
- If Home Assistant state is unavailable, show the last known state cautiously, or show "unavailable" rather than display false success.
- Update the UI optimistically only when safe (for example, the brightness arc may move with the encoder immediately), then reconcile with the Home Assistant state when it returns.
- If a command confirmation fails, avoid flashing or noisy error behavior; a small inline indicator is enough.

### Bedside behavior notes (when HA is unavailable)

- Local gesture detection can still happen even if Home Assistant is unavailable; the ESP32 sees and decodes the gesture.
- An optional status LED can acknowledge "gesture detected" locally so the user knows the input registered, independent of whether the bulb action eventually succeeds.
- Do not imply that bulbs changed unless the Home Assistant command succeeded. The status LED is about input registration, not bulb confirmation.
- Cooldowns remain active even during Home Assistant failure to prevent command spam from rapid gesture re-tries.

### Manual wall switch

- A manual wall switch on the bulb circuit is the only simple true "everything is broken" fallback. It works regardless of which part of the stack has failed.
- The wall switch is outside firmware and outside the app flow. It lives at the wall.
- Relay or mains-control hardware on the ESP32 is **not** part of the first-build firmware.

## Error handling

- Ignore unclear gestures instead of guessing.
- Do not flash lights on noisy sensor readings.
- If Home Assistant API is unavailable, fail quietly and recover automatically when the connection returns.
