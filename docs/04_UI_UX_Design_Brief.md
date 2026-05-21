# Document 04 — UI / UX Design Brief

## Direction

VelaDial should feel like a small, quiet bedside or door-side instrument rather than a flashy dashboard. The display is round, small, and used in a dark bedroom, so the interface should be minimal and highly legible.

## Visual style

- Dark-mode only.
- Pure black background.
- Amber accent color.
- White text for high contrast.
- Large readable typography.
- Simple round-screen geometry.
- Minimal motion.
- No bright full-screen white states.
- Performance-safe for ESPHome LVGL.

## Color palette

| Purpose | Color | Notes |
| --- | --- | --- |
| Background | `#000000` | Helps blend into the black bezel. |
| Primary active | `#FFB300` | Warm amber, used for brightness and active glow. |
| Text | `#FFFFFF` | Keep high contrast. |
| On state | `#4CAF50` | Use sparingly. |
| Off/error state | `#F44336` | Use sparingly; avoid alarm-like visuals. |

## Typography

- Use Roboto or a similarly clean sans-serif font.
- Primary values should be large enough to read quickly.
- Avoid long labels; the display is too small for paragraph UI.

## Main UI pages

The main door-side UI has exactly 3 primary pages:

1. Power
2. Brightness
3. Presets

## Interaction model

### Rotary input

The physical rotation should feel directly connected to the brightness arc. The display should update immediately, even if the Home Assistant command is debounced slightly.

### Touch input

Touch should be reserved for major actions:

- Wake screen.
- Toggle power.
- Select preset.
- Move between pages.

### Sleep behavior

The display should dim after inactivity. Waking the display should not accidentally change light state unless the user performs a deliberate second action.

## UI screens

### Power

- Large central power symbol or state label.
- Small status text: On / Off / Offline.
- Optional amber ring when lights are on.

### Brightness

- Arc or circular progress indicator.
- Large percentage value in the center.
- Rotary encoder is the primary input.

### Presets

- Few large touch targets.
- No dense grid.
- Clear active preset indication.

## Ambient-aware display behavior

TSL2591 controls adaptive backlight behavior on the door-side display.

- Display should be brighter during daylight/bright room conditions.
- Display should be very dim in dark bedroom conditions.
- Backlight transitions must be smooth, not flickery.
- Ambient light affects display/backlight only, not room bulbs directly.
- Night safety is more important than maximum brightness.
- First interaction may wake the display, but night brightness should still be capped.

## Day / dim / night UI modes

| Mode | Trigger | Backlight | UI Behavior |
| --- | --- | --- | --- |
| Bright | TSL2591 lux > 50 | High | Normal contrast, readable from door-side distance. |
| Dim | TSL2591 lux 5–50 | Moderate | Amber/white contrast preserved. |
| Night | TSL2591 lux < 5 | Low | Minimal glow, no large bright fills. |

The UI layout should not change dramatically between modes; only brightness/intensity should adapt.

## LED ring brightness behavior

- LED ring should stay off when room lights are off.
- When lights are on, LED ring may glow amber at low brightness.
- LED ring brightness may be capped by ambient-light mode.
- No flashing error patterns at night.
- No aggressive animations.
- Optional future: ambient-aware LED ring intensity.

## Bedside sensor UX

- VL53L4CD hand-near detection should feel intentional, not jumpy.
- APDS-9960 still owns left/right gestures.
- Nightlight hold must feel deliberate: stable hand at 5–10 cm for about 1.5 s.
- Quick pass-by motion must not trigger nightlight.
- Cooldowns should prevent repeated actions.
- No audio, no voice, no beeps.

## Environmental data UX

SHT45 temperature/humidity is secondary.

- Do not place temp/humidity on the main Power/Brightness/Presets pages as a required element.
- Environmental data can appear later in:
  - diagnostics/info view
  - idle mini-status
  - long-press info screen
  - Home Assistant dashboard
- Environmental data must not compete with lighting controls.
- No HVAC/comfort automation UX is required for the first build.

## Wake and accidental-trigger UX

- Touch and knob rotation while asleep should wake first.
- Physical knob press while asleep remains a design decision to test.
- Avoid accidental room-light changes from bumps.
- Bedroom safety is more important than shortcut speed.
- Sensor readings must not cause sudden full-brightness room light changes.

## Fallback UX

- If TSL2591 fails, use fixed dim/sleep behavior.
- If SHT45 fails, hide environmental data.
- If VL53L4CD fails, fall back to APDS-9960 gesture-only behavior if possible.
- If APDS-9960 fails, ToF nightlight hold may still work if VL53L4CD works.
- Fail quietly and recover automatically.
- Show errors only in diagnostics, not with flashing LEDs.

## LVGL feasibility guidance

- Prefer simple arcs, labels, circles, and large buttons.
- Avoid tiny widgets and dense dashboard layouts.
- Avoid complex 3D, heavy gradients, and excessive animation in final firmware.
- AI-generated visual concepts can inspire the design, but final UI must be practical for ESPHome LVGL on a 240x240 round display.

## Design references

- Smart thermostat rotary control clarity.
- Home Assistant dark theme simplicity.
- SmartKnob-style relationship between physical dial and visual feedback.

These are references, not skins to copy exactly.
