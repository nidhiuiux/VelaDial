# Document 04 — UI / UX Design Brief

## Direction

VelaDial should feel like a small, quiet bedside or door-side instrument rather than a flashy dashboard. The display is round, small, and used in a dark bedroom, so the interface should be minimal and highly legible.

## Visual style

- Dark-mode only.
- Large typography for power state and brightness percentage.
- Simple geometry that works on a 240 x 240 round screen.
- Limited motion; avoid anything that looks busy at night.
- No bright full-screen white states.

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

## Design references

- Smart thermostat rotary control clarity.
- Home Assistant dark theme simplicity.
- SmartKnob-style relationship between physical dial and visual feedback.

These are references, not skins to copy exactly.
