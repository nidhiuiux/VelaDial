# Interaction Inventory — ELECROW 1.28" Rotary Display

This document catalogs every possible physical interaction the ELECROW 1.28" rotary display hardware can support, classifies each for the VelaDial first build, and explains the reasoning.

---

## Classification Key

| Classification | Meaning |
| :--- | :--- |
| **Recommended** | Should be implemented in the first production build. Essential for core functionality. |
| **Optional / Future** | Useful but not critical. Can be added after the first build is stable and tested. |
| **Avoid** | Introduces complexity, accidental triggers, or unreliable behavior. Not suitable for first build. |

---

## 1. Physical Knob Interactions

### 1.1 Rotate Clockwise (CW)

**Classification: Recommended**

This is the primary input for increasing brightness. The rotary encoder on GPIO 45/42 produces detent-based steps. Each clockwise click should increase brightness by 5% on the Brightness page. The physical rotation must feel directly connected to the on-screen arc filling up.

**Why Recommended:** This is the core value proposition of a rotary display controller. It is the most natural, silent, and precise way to adjust brightness without looking at a phone.

---

### 1.2 Rotate Counter-Clockwise (CCW)

**Classification: Recommended**

The inverse of CW rotation. Each counter-clockwise click decreases brightness by 5%. The on-screen arc should empty correspondingly.

**Why Recommended:** Inseparable from CW rotation. Together they form the fundamental brightness control loop.

---

### 1.3 Short Press (Single Click)

**Classification: Recommended**

A brief press and release of the rotary encoder button (GPIO 41). In the current bring-up firmware, this toggles the light group. In the full UI, its behavior should be context-dependent:

- On the Power page: toggle the bedroom light group on/off.
- On the Brightness page: confirm the current brightness (optional) or toggle power.
- On the Preset page: confirm the selected preset (optional) or toggle power.

For the first build, a consistent behavior is safest: **short press always toggles power**, regardless of which page is active. This prevents confusion.

**Why Recommended:** The knob press is the fastest, most intuitive "light switch" replacement. It requires zero visual attention and works even when the screen is asleep.

---

### 1.4 Long Press (Hold > 1 second)

**Classification: Optional / Future**

A press held for more than approximately 1 second before release. Possible uses:

- Return to the Power (home) page from any other page.
- Enter a "settings" or "sleep" mode.
- Activate nightlight preset directly.

**Why Optional:** Long press adds a useful shortcut but requires careful timing thresholds. If the threshold is too short, normal presses become ambiguous. If too long, it feels unresponsive. Better to add after the basic UI is stable and the user can tune the threshold on real hardware.

---

### 1.5 Double Press (Two clicks within ~400ms)

**Classification: Avoid**

Two rapid presses of the rotary button.

**Why Avoid:** Double-press detection on a physical rotary encoder is unreliable. The mechanical bounce of the switch makes it difficult to distinguish a double press from two separate single presses. It also introduces a mandatory delay on single-press detection (the system must wait to see if a second press follows), which makes the UI feel sluggish. Not suitable for a bedroom controller where speed and reliability matter.

---

### 1.6 Press + Rotate (Hold button while turning)

**Classification: Avoid**

Holding the encoder button down while simultaneously rotating the knob.

**Why Avoid:** This is physically awkward on a small rotary knob. The user must press down and twist at the same time, which requires two-finger dexterity. It is also difficult to implement reliably in ESPHome without custom C++ code. The added complexity is not justified when touch + knob already provides a richer interaction model.

---

### 1.7 Fast Rotate (Multiple detents per second)

**Classification: Recommended**

Spinning the knob quickly through many detents in rapid succession.

The UI should handle this gracefully:
- Update the visual arc immediately on every detent (local rendering).
- Debounce the Home Assistant command (send the final value after rotation stops for ~300ms).

This prevents flooding the network with 20 brightness commands per second while still feeling responsive on-screen.

**Why Recommended:** Users will naturally spin the knob fast to go from 10% to 90%. The system must handle this without lag, dropped frames, or command flooding.

---

### 1.8 Slow Rotate (One detent at a time)

**Classification: Recommended**

Turning the knob carefully, one click at a time, for fine-grained adjustment.

Each detent should produce exactly one 5% brightness step. The arc and percentage label must update on every single detent without delay.

**Why Recommended:** Fine-tuning brightness is a primary use case, especially at night when the user wants to go from 10% to 15% without overshooting.

---

### 1.9 Detent-Based Step Interaction

**Classification: Recommended**

The encoder has physical detents (click stops). Each detent equals one logical step. The UI should map 1 detent = 1 increment (5% brightness, or 1 preset position on the color page).

**Why Recommended:** Detent-based stepping is what makes a physical knob feel precise and predictable. It is the hardware's native behavior and should be respected, not smoothed over.

---

### 1.10 Accidental Bump Handling

**Classification: Recommended**

The knob may be accidentally bumped when walking past the door-side controller. The system must prevent accidental light changes:

- If the screen is asleep (dimmed to 10%), the first knob movement should ONLY wake the screen, not change brightness or toggle power.
- Only after the screen is awake should subsequent knob movements adjust brightness.
- A single accidental detent while the screen is asleep should not change anything.

**Why Recommended:** This is a critical safety behavior for bedroom use. Without it, bumping the controller while walking past at night could flash the lights and wake a sleeping person.

---

## 2. Touch Interactions

### 2.1 Single Tap (Center)

**Classification: Recommended**

A quick tap on the center area of the round screen.

- On the Power page: toggle lights on/off.
- On the Brightness page: could toggle power or do nothing (knob is primary here).
- On the Preset page: no action (presets have their own tap targets).

For the first build, center tap on the Power page toggles lights. On other pages, center tap returns to the Power page.

**Why Recommended:** Tapping the center of a round screen is the most natural touch gesture. It serves as the primary "power button" equivalent.

---

### 2.2 Long Touch (Hold > 1 second)

**Classification: Optional / Future**

Pressing and holding a finger on the screen for more than 1 second.

Possible uses:
- Activate nightlight mode directly.
- Enter an "edit presets" mode.
- Show additional information (bulb status, WiFi signal).

**Why Optional:** Long touch is useful but not essential for the first build. The core flows (power, brightness, presets) are already covered by tap and knob. Adding long touch later provides a natural extension point.

---

### 2.3 Swipe Left / Right Between Pages

**Classification: Recommended**

A horizontal swipe gesture to navigate between the three pages (Power ↔ Brightness ↔ Presets).

- Swipe left: move to the next page.
- Swipe right: move to the previous page.

LVGL natively supports swipeable page containers (tileview or tabview), making this straightforward to implement.

**Why Recommended:** Swiping is the standard navigation paradigm for multi-page round displays (smartwatches, car infotainment knobs). Users will expect it. It keeps the UI clean by avoiding permanent navigation buttons that consume precious screen space.

---

### 2.4 Circular Swipe Around the Screen Edge

**Classification: Avoid**

Dragging a finger in a circular motion along the outer edge of the round display to adjust a value (like a virtual dial).

**Why Avoid:** On a 240x240 display (approximately 32mm diameter viewable area), the outer edge is extremely narrow. Finger tracking at the edge is unreliable because:
- The physical bezel occludes part of the touch area.
- The CST816D touch controller has reduced accuracy at screen edges.
- The user's finger is larger than the arc track width.

The physical rotary knob already provides this exact interaction more reliably. Duplicating it with a less reliable touch gesture adds confusion without benefit.

---

### 2.5 Touch-and-Drag on Brightness Arc

**Classification: Optional / Future**

Touching the brightness arc widget and dragging along it to set brightness directly (like a slider).

**Why Optional:** While LVGL supports this natively on arc widgets, the physical knob is the primary brightness input for VelaDial. Adding touch-drag on the arc creates a secondary input path that could conflict with swipe navigation (the system must distinguish "is the user swiping to change pages or dragging the arc?"). Better to add after the knob-based flow is proven stable.

---

### 2.6 Tap Preset Buttons

**Classification: Recommended**

Tapping one of 4-6 large color preset buttons on the Preset page.

Each button should be large enough to tap reliably on the small screen (minimum 50x50px touch target). On tap, the system sends the corresponding color/temperature command to Home Assistant.

**Why Recommended:** Preset selection is a core feature defined in the App Flow document. Touch is the natural input for selecting from a discrete set of options (as opposed to continuous adjustment, which is better suited to the knob).

---

### 2.7 Tap Edge Navigation Zones

**Classification: Avoid**

Dedicating thin strips at the left/right edges of the screen as invisible "next page" / "previous page" tap targets.

**Why Avoid:** On a 240px-wide round screen, edge zones would be approximately 20-30px wide. This is too narrow for reliable finger targeting, especially in the dark. Swipe gestures are more forgiving and do not consume visible screen real estate.

---

### 2.8 Wake-Only First Touch

**Classification: Recommended**

When the screen is in its dimmed idle state (10% backlight), the first touch interaction should ONLY wake the screen to full brightness. It should NOT trigger any action (no power toggle, no page change, no preset selection).

Only the second deliberate interaction (after the screen is visibly awake) should perform an action.

**Why Recommended:** This is explicitly required by the UI/UX Design Brief and App Flow documents. It prevents accidental light changes when the user brushes the screen or when the controller is bumped. Critical for bedroom use.

---

### 2.9 Touch Lock / Sleep Mode Behavior

**Classification: Optional / Future**

A mode where touch is completely disabled (only the knob works), or where the screen is fully off and requires a specific gesture to wake.

**Why Optional:** The current 60-second idle dim + wake-only-first-touch behavior is sufficient for the first build. A full touch lock adds complexity (how does the user unlock it?) without solving a problem that the wake-only behavior doesn't already address.

---

## 3. Combined Interactions

### 3.1 Touch Screen to Select Mode, Rotate Knob to Adjust

**Classification: Recommended**

The fundamental interaction model for VelaDial:
1. User swipes to the Brightness page (touch).
2. User turns the knob to adjust brightness (rotary).
3. User swipes to the Preset page (touch).
4. User taps a preset (touch).

Touch handles navigation and selection. Knob handles continuous adjustment. They complement each other without conflict.

**Why Recommended:** This is the cleanest separation of concerns. It leverages each input method for what it does best and avoids ambiguity.

---

### 3.2 Press Knob to Confirm

**Classification: Optional / Future**

Using the knob press as a "confirm" or "apply" action after making a selection.

**Why Optional:** In the first build, actions should be immediate. Turning the knob changes brightness instantly (after debounce). Tapping a preset applies it instantly. Adding a "confirm" step slows down the interaction and adds cognitive load. However, it could be useful in future features like "create custom preset" where confirmation prevents accidental saves.

---

### 3.3 Long Press Knob to Return Home

**Classification: Optional / Future**

Holding the knob for 1+ seconds to jump back to the Power page from any other page.

**Why Optional:** Useful as a shortcut, but swiping back is already fast on a 3-page UI. Adding this after the basic navigation is stable provides a nice power-user shortcut without complicating the first build.

---

### 3.4 Rotate While on Different Pages

**Classification: Recommended**

The knob's behavior should be context-aware based on the active page:

- **Power page:** Rotation wakes the screen (if asleep) but does NOT toggle power. This prevents accidental bumps from changing light state.
- **Brightness page:** Rotation adjusts brightness (primary use).
- **Preset page:** Rotation could scroll through presets (optional) or do nothing (simpler).

For the first build, the safest approach:
- Power page: rotation wakes only.
- Brightness page: rotation adjusts brightness.
- Preset page: rotation does nothing (presets are tap-only).

**Why Recommended:** Context-aware knob behavior prevents the most dangerous failure mode: accidentally changing lights from the Power page by bumping the knob.

---

### 3.5 Touch Preset, Rotate to Fine-Tune

**Classification: Avoid**

Tapping a preset to select it, then using the knob to fine-tune the color temperature or brightness of that specific preset.

**Why Avoid:** This creates a complex stateful interaction where the knob's meaning changes based on what was last tapped. It is confusing without clear visual feedback and difficult to implement cleanly in LVGL. The Brightness page already handles brightness adjustment. Presets should be "tap and done."

---

### 3.6 Press to Toggle, Rotate to Brightness (Single-Page Mode)

**Classification: Optional / Future**

A simplified single-page mode where:
- Press = toggle power.
- Rotate = adjust brightness.
- No page navigation needed.

**Why Optional:** This is actually a compelling "minimal mode" that could replace the 3-page UI for users who only need power and brightness. However, it eliminates color presets entirely. Better to build the full 3-page UI first, then consider offering this as an alternative "simple mode" later.

---

## 4. LED Ring Interactions

### 4.1 Amber Glow When Lights Are On

**Classification: Recommended**

When the bedroom lights are on, the 5 WS2812 LEDs glow a soft amber (#FFB300) at low brightness (approximately 10-20% LED power). This provides a subtle ambient indicator that the lights are active, even when the screen is dimmed.

**Why Recommended:** The LED ring is the only feedback visible from across the room when the screen is asleep. It answers the question "are my lights on?" without needing to walk over and wake the screen.

---

### 4.2 Dim Pulse on Wake

**Classification: Optional / Future**

A brief, gentle pulse animation on the LED ring when the screen wakes from idle. This provides tactile-visual feedback that the device acknowledged the touch/knob input.

**Why Optional:** Nice feedback, but adds animation complexity. The screen brightening from 10% to 100% already provides clear wake feedback. The LED pulse can be added later as a polish item.

---

### 4.3 Off When Lights Are Off

**Classification: Recommended**

When the bedroom lights are off, the LED ring should be completely off. No glow, no pulse, no standby indicator. In a dark bedroom, even a faint LED is visible and potentially annoying.

**Why Recommended:** Zero light pollution when lights are off is a core design principle. The LED ring must not become a nightlight by itself.

---

### 4.4 Brief Color Preview When Preset Selected

**Classification: Optional / Future**

When the user taps a color preset (e.g., "Blue"), the LED ring briefly flashes that color for 1-2 seconds before returning to amber (or off). This gives immediate local feedback before the bulbs respond.

**Why Optional:** Visually satisfying and provides faster perceived feedback than waiting for the bulbs. However, it requires mapping preset colors to WS2812 values and adds code complexity. Good for a second iteration.

---

### 4.5 Error / Offline Indication

**Classification: Avoid**

Using the LED ring to indicate errors (e.g., Home Assistant disconnected, WiFi lost) by flashing red or pulsing.

**Why Avoid:** In a bedroom, any unexpected flashing light at night is unacceptable. If the system goes offline, it should fail silently. The user will notice when their next command does not work. Error states can be shown on-screen (small icon) when the user actively wakes the display, but never via the LED ring unprompted.

---

## 5. Summary Table

| # | Interaction | Classification | Primary Reason |
| :--- | :--- | :--- | :--- |
| 1.1 | Rotate CW | Recommended | Core brightness increase |
| 1.2 | Rotate CCW | Recommended | Core brightness decrease |
| 1.3 | Short press | Recommended | Primary power toggle |
| 1.4 | Long press | Optional / Future | Useful shortcut, needs tuning |
| 1.5 | Double press | Avoid | Unreliable detection, adds latency |
| 1.6 | Press + rotate | Avoid | Physically awkward, complex code |
| 1.7 | Fast rotate | Recommended | Must handle gracefully with debounce |
| 1.8 | Slow rotate | Recommended | Fine-grained adjustment |
| 1.9 | Detent steps | Recommended | Native hardware behavior |
| 1.10 | Accidental bump | Recommended | Critical bedroom safety |
| 2.1 | Tap center | Recommended | Primary touch action |
| 2.2 | Long touch | Optional / Future | Extension point |
| 2.3 | Swipe L/R | Recommended | Page navigation |
| 2.4 | Circular swipe | Avoid | Unreliable on small screen |
| 2.5 | Drag on arc | Optional / Future | Conflicts with swipe |
| 2.6 | Tap presets | Recommended | Core preset selection |
| 2.7 | Edge tap zones | Avoid | Too narrow, unreliable |
| 2.8 | Wake-only first touch | Recommended | Prevents accidental triggers |
| 2.9 | Touch lock | Optional / Future | Not needed with wake-only |
| 3.1 | Touch nav + knob adjust | Recommended | Core interaction model |
| 3.2 | Press to confirm | Optional / Future | Slows interaction |
| 3.3 | Long press = home | Optional / Future | Nice shortcut, not essential |
| 3.4 | Context-aware rotation | Recommended | Prevents accidental changes |
| 3.5 | Preset + fine-tune | Avoid | Confusing stateful behavior |
| 3.6 | Single-page mode | Optional / Future | Good alternative, not first build |
| 4.1 | Amber glow (on) | Recommended | Ambient status indicator |
| 4.2 | Pulse on wake | Optional / Future | Polish item |
| 4.3 | Off when lights off | Recommended | Zero light pollution |
| 4.4 | Color preview flash | Optional / Future | Nice feedback, adds complexity |
| 4.5 | Error flash | Avoid | Unacceptable in bedroom at night |
