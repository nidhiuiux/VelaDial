# UI Concept Direction Matrix — ELECROW 1.28" Rotary Display

This document explores 12 distinct UI direction concepts for the VelaDial door-side controller. It evaluates how each concept utilizes the 240x240 round display, touch, rotary knob, and LED ring, while respecting the constraints of a dark, silent, local-first bedroom environment.

---

## Open Design Decision: Sleep State Wake Behavior

Before evaluating concepts, we must track an open design decision regarding the sleep state (when the screen is dimmed to 10% after 60s of inactivity):

**"Should a short press of the physical knob toggle power while asleep, or should it wake the screen first like touch/rotation?"**

*   **Option A (Wake-Only):** Safest. Prevents accidental light flashes if bumped. Requires two actions to turn on lights (press to wake, press again to toggle).
*   **Option B (Instant Toggle):** Acts like a traditional light switch. Fastest interaction. Higher risk of accidental triggers.

This decision will be finalized after visual mockups and initial user testing.

---

## Concept 1: Minimal Thermostat Style

*   **Short Description:** Inspired by Nest thermostats. A single large number dominates the screen, with subtle status indicators around the edge.
*   **Main Screen Layout:** Huge center text (e.g., "75%"). Small power icon at the bottom.
*   **Page Structure:** 3-page (Power / Brightness / Presets).
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Adjusts the large center number (brightness).
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe to change pages. Tap center on Power page to toggle.
*   **LED Ring:** Amber glow when on.
*   **Pros:** Extremely legible from a distance. Clean, familiar aesthetic.
*   **Cons:** Wastes screen real estate. Hard to fit preset buttons elegantly.
*   **Risk on 240x240:** Low. Large text renders well.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 2: SmartKnob-Inspired Arc UI

*   **Short Description:** Focuses on the relationship between physical rotation and visual feedback using a thick, colorful edge arc.
*   **Main Screen Layout:** Thick amber arc hugging the outer edge. Center shows percentage or power state.
*   **Page Structure:** 3-page (Power / Brightness / Presets).
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Fills/empties the arc.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe to navigate.
*   **LED Ring:** Mirrors the arc's fill level (if possible) or solid amber.
*   **Pros:** Highly tactile feel. Excellent visual feedback for rotation.
*   **Cons:** Thick arcs can look pixelated on low-res screens if not anti-aliased well.
*   **Risk on 240x240:** Medium. Requires careful LVGL arc styling.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 3: Large Center Power Button (Power-Core)

*   **Short Description:** Prioritizes the most common action (turning lights on/off) with a massive, unmissable touch target.
*   **Main Screen Layout:** A giant circular button fills 80% of the screen.
*   **Page Structure:** 3-page, but Power page is the absolute focus.
*   **Best Interaction Model:** Touch-first for power, knob for secondary.
*   **Knob Rotation:** Wakes screen on Power page; adjusts brightness on Brightness page.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap the giant center button.
*   **LED Ring:** Solid amber when on.
*   **Pros:** Impossible to miss the touch target. Very satisfying to tap.
*   **Cons:** Less elegant for brightness/presets.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 4: Single-Page Simple Mode

*   **Short Description:** Abandons the 3-page swipe model entirely. Everything is on one screen.
*   **Main Screen Layout:** Center power icon, surrounded by a thin brightness arc. No presets.
*   **Page Structure:** 1-page only.
*   **Best Interaction Model:** Press to toggle, rotate to dim.
*   **Knob Rotation:** Always adjusts brightness.
*   **Knob Press:** Always toggles power.
*   **Touch:** Tap center to toggle.
*   **LED Ring:** Amber when on.
*   **Pros:** Zero navigation required. Lowest cognitive load.
*   **Cons:** Sacrifices color presets entirely.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 5: Preset Ring UI

*   **Short Description:** Focuses on mood lighting. Presets are arranged in a circle around the edge, with the center showing the active state.
*   **Main Screen Layout:** 4-6 colored dots around the perimeter. Center shows power/brightness.
*   **Page Structure:** 2-page (Main / Brightness).
*   **Best Interaction Model:** Touch for presets, knob for brightness.
*   **Knob Rotation:** Adjusts brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap the perimeter dots to change color.
*   **LED Ring:** Briefly flashes the selected preset color.
*   **Pros:** Fast access to moods without swiping.
*   **Cons:** Touch targets on the perimeter might be too small/close to the bezel.
*   **Risk on 240x240:** High. Small touch targets on a 1.28" screen are error-prone.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes (to test target size viability).

## Concept 6: Night Mode Ultra-Minimal

*   **Short Description:** Designed specifically for zero light pollution. Almost entirely black, using only thin outlines and dark amber.
*   **Main Screen Layout:** Wireframe icons only. No solid fills.
*   **Page Structure:** 3-page.
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Adjusts brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe to navigate.
*   **LED Ring:** Disabled entirely, or extremely dim (1%).
*   **Pros:** Perfect for sensitive sleepers.
*   **Cons:** Might be too hard to read during the day.
*   **Risk on 240x240:** Medium. Thin lines can look jagged.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 7: Text-First Utility UI

*   **Short Description:** Abandons icons in favor of clear, large text labels (e.g., "ON", "OFF", "WARM").
*   **Main Screen Layout:** Stacked text. Top: "Bedroom". Middle: "ON". Bottom: "75%".
*   **Page Structure:** 3-page.
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Adjusts brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe to navigate.
*   **LED Ring:** Amber when on.
*   **Pros:** Zero ambiguity. Very readable.
*   **Cons:** Can look utilitarian or boring.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 8: Apple Watch-Style Complications

*   **Short Description:** Multiple small data points arranged in corners/edges (e.g., brightness top left, active preset bottom right).
*   **Main Screen Layout:** Center power, small icons in the "corners" of the circle.
*   **Page Structure:** 1-page or 2-page.
*   **Best Interaction Model:** Touch targets for different functions.
*   **Knob Rotation:** Adjusts whatever was last touched.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap complications to activate them.
*   **LED Ring:** Amber when on.
*   **Pros:** Data-dense.
*   **Cons:** Violates the "large readable labels" constraint. Too cluttered for a quick door-side glance.
*   **Risk on 240x240:** High. Clutter and tiny touch targets.
*   **ESPHome/LVGL Feasibility:** Hard (complex layout).
*   **Include in Visual Batch:** No (violates core constraints).

## Concept 9: Ambient LED-Ring Status-First

*   **Short Description:** Relies heavily on the physical LED ring for feedback, keeping the screen mostly off or very minimal.
*   **Main Screen Layout:** Blank or just a tiny power dot.
*   **Page Structure:** 1-page.
*   **Best Interaction Model:** Knob-heavy.
*   **Knob Rotation:** Adjusts brightness, LED ring fills up physically to match.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap to wake.
*   **LED Ring:** Acts as the primary brightness/status indicator.
*   **Pros:** Very physical, hardware-centric feel.
*   **Cons:** The ELECROW only has 5 LEDs in the ring, making it too low-resolution to act as a smooth brightness gauge.
*   **Risk on 240x240:** High (hardware limitation of 5 LEDs).
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** No (hardware doesn't support it well).

## Concept 10: Three-Screen Tab Carousel

*   **Short Description:** The standard approach defined in the App Flow, but with visible navigation dots at the bottom to show where the user is.
*   **Main Screen Layout:** Content in center, 3 small dots at the bottom edge.
*   **Page Structure:** 3-page (Power / Brightness / Presets).
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Context-aware (wakes on Power, adjusts on Brightness).
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe to navigate.
*   **LED Ring:** Amber when on.
*   **Pros:** Familiar mobile OS pattern. User never gets lost.
*   **Cons:** Tab dots take up vertical space.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Easy (native LVGL tabview).
*   **Include in Visual Batch:** Yes.

## Concept 11: Brightness-First UI

*   **Short Description:** Assumes the user mostly wants to dim the lights. The default screen is the brightness arc.
*   **Main Screen Layout:** Brightness arc is always visible. Power toggle is a smaller button inside the arc.
*   **Page Structure:** 2-page (Brightness+Power / Presets).
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Always adjusts brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap center to toggle, swipe for presets.
*   **LED Ring:** Amber when on.
*   **Pros:** Removes one swipe step for the most common rotary action.
*   **Cons:** Makes the power toggle visually secondary.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes.

## Concept 12: Door Switch Replacement UI

*   **Short Description:** Mimics a physical Decora rocker switch visually.
*   **Main Screen Layout:** Split vertically. Top half = ON, Bottom half = OFF.
*   **Page Structure:** 2-page (Switch / Brightness).
*   **Best Interaction Model:** Touch-heavy.
*   **Knob Rotation:** Adjusts brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap top to turn on, tap bottom to turn off.
*   **LED Ring:** Amber when on.
*   **Pros:** Instantly understandable to guests.
*   **Cons:** Doesn't fit a round screen well. Wastes the rotary knob's potential.
*   **Risk on 240x240:** Medium (awkward layout).
*   **ESPHome/LVGL Feasibility:** Easy.
*   **Include in Visual Batch:** Yes (as a contrasting option).
