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

### Page navigation

- Horizontal swipe navigates between pages.
- Left swipe = next page (Power → Brightness → Presets → Power).
- Right swipe = previous page.
- A 3-dot page indicator appears near the bottom of the round screen.
- The active page's dot is amber; inactive dots are low-contrast white.
- There is **no** 4th Environment page in the first build.

### Knob press behavior per page

Knob press is page-specific and applies only when the display is awake:

- **Power page:** knob press toggles the bedroom light group (same as tapping the center).
- **Brightness page:** knob press returns to the Power page.
- **Presets page:** knob press applies the currently highlighted preset.
- **While asleep:** knob press wakes only.

## UI screens

### Power

- Large central power symbol or state label.
- Small status text: On / Off / Unavailable.
- Optional amber ring when lights are on.

### Brightness

- Arc or circular progress indicator.
- Large percentage value in the center.
- Rotary encoder is the primary input.

### Presets

First-build presets are locked to exactly 4:

1. Warm White
2. Soft Amber
3. Neutral White
4. Low Nightlight

Layout:

- 2×2 large touch targets on the 240×240 round screen.
- Each tile sized for thumb-tap; tiles stay inside the inscribed area so corners are not cut off by the round bezel.
- Active preset is highlighted with an amber border (or equivalent amber highlight).

Out of first-build scope:

- Optional RGB accent preset is future / v2 only.
- No dense color picker, color wheel, or per-bulb editor in first build.

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

This section describes the **door-side WS2812 LED ring** built into the ELECROW rotary display module. It is a separate system from the optional **bedside status LED** described in `docs/03_App_Flow.md` and is independently controlled.

- LED ring should stay off when room lights are off.
- When lights are on, LED ring may glow amber at low brightness.
- LED ring brightness may be capped by ambient-light mode.
- No flashing error patterns at night.
- No aggressive animations.
- Optional future: ambient-aware LED ring intensity.

## Bedside sensor UX

The bedside controller has no screen, so its UX is entirely physical sensors plus an optional silent LED. v1 and v2 are split to match `docs/03_App_Flow.md`.

### v1 (first build)

- **APDS-9960 standalone left/right gestures.** APDS-9960 owns directional gestures without being armed or gated by VL53L4CD.
- **VL53L4CD standalone hand-hold nightlight** (if VL53L4CD ESPHome support is verified). Nightlight hold must feel deliberate: stable hand at 5–10 cm for about 1.5 s.
- Quick pass-by motion must not trigger nightlight.
- Cooldowns prevent repeated actions.
- No audio, no voice, no beeps.

### v2 / future (not required for first firmware)

- VL53L4CD arms, wakes, or refines APDS-9960 gesture detection (sensor fusion).
- Sensor fusion is **not** required for the first firmware build. It should be added only after both v1 standalone paths are verified to work reliably in the actual bedside mounting.

## Environmental data UX

SHT45 temperature/humidity is secondary.

- Do not place temp/humidity on the main Power / Brightness / Presets pages as a required element.
- There is **no** required 4th Environment page in the first build.
- A diagnostics view, idle mini-status, or long-press info overlay are **future / v2 possibilities only, not first build**.
- Environmental data is exposed to Home Assistant and can be shown in the Home Assistant dashboard.
- Environmental data must not compete with lighting controls.
- No HVAC / comfort automation UX is required for the first build.

## Wake and accidental-trigger UX

The following wake behavior is locked for the first build (matches `docs/03_App_Flow.md`):

- **Touch while asleep:** wake only.
- **Knob rotation while asleep:** wake only.
- **Knob press while asleep:** wake only.
- A second deliberate action while the display is awake is what toggles or sends a command.
- Avoid accidental room-light changes from bumps.
- Bedroom safety is more important than shortcut speed.
- Sensor readings must not cause sudden full-brightness room light changes.

## Offline / unavailable visual state

When Home Assistant, the Raspberry Pi, or the local LAN is unreachable, the door-side display should clearly indicate the state without aggressive UX. This matches the connectivity behavior in `docs/03_App_Flow.md`.

- Use **"Unavailable"** as the preferred UI wording on the door-side display.
- Show a subtle, persistent badge or small label rather than a full-screen overlay.
- Do **not** pretend a command succeeded without state confirmation from Home Assistant.
- Dim or desaturate affected controls if useful for clarity (for example, gray the brightness percentage, mute the power-state icon).
- **No** flashing.
- **No** popup modal.
- **No** audio.
- **No** full-screen aggressive error.
- The offline / unavailable state belongs mainly on the door-side display, not on a constantly lit bedside LED.

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
