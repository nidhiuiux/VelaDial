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

## Concept 13: Tidal/Lunar Phase Visualization (Novelty: 9)

*   **Short Description:** Uses the moon phase as a natural, intuitive metaphor for brightness level on a round display.
*   **Main Screen Layout:** A central moon graphic that transitions through its phases (new moon to full moon).
*   **Page Structure:** 2-page (Lunar Brightness / Presets).
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Adjusts the moon phase. A new moon represents 0% brightness (off), while a full moon represents 100%.
*   **Knob Press:** Toggles power (jumps between new and full moon).
*   **Touch:** Swipe to access presets.
*   **LED Ring:** Amber glow when on.
*   **Pros:** Extremely intuitive and calming. Perfect for a bedroom environment. No numbers needed.
*   **Cons:** Harder to set an exact percentage if desired.
*   **Risk on 240x240:** Low. A moon graphic scales well.
*   **ESPHome/LVGL Feasibility:** Medium (requires pre-rendered image sequence or custom drawing).
*   **Include in Visual Batch:** Yes.

## Concept 14: Sundial Shadow UI (Novelty: 9)

*   **Short Description:** A time-of-day aware UI where a digitally rendered central gnomon casts a dynamic shadow that rotates based on the time of day.
*   **Main Screen Layout:** A central "pin" casting a shadow across the circular face.
*   **Page Structure:** 1-page (Contextual).
*   **Best Interaction Model:** Touch nav + knob adjust.
*   **Knob Rotation:** Adjusts brightness, but the baseline is set by the "time" indicated by the shadow.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap to override the time-based suggestion.
*   **LED Ring:** Amber when on.
*   **Pros:** Highly contextual and unique. Connects indoor lighting to the natural world.
*   **Cons:** Might be confusing if the user just wants a simple dimmer.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 15: Tree Ring Growth Pattern (Novelty: 9)

*   **Short Description:** Concentric rings radiating from the center, mimicking a tree cross-section, where rings represent saved scenes or brightness history.
*   **Main Screen Layout:** Organic concentric rings.
*   **Page Structure:** 2-page (Rings / Presets).
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Expands or contracts the rings to adjust brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Tap specific rings to recall past states.
*   **LED Ring:** Amber when on.
*   **Pros:** Organic, non-technical aesthetic.
*   **Cons:** Might be too abstract for quick use.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 16: Topographic Contour Map (Novelty: 9)

*   **Short Description:** Elevation lines represent brightness levels on a circular terrain.
*   **Main Screen Layout:** Stylized topographic map lines.
*   **Page Structure:** 2-page.
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** "Climbs" or "descends" the terrain. As the dial turns clockwise, the UI highlights higher elevation lines (brighter).
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe for presets.
*   **LED Ring:** Amber when on.
*   **Pros:** Visually striking and unique.
*   **Cons:** Abstract representation of brightness.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 17: Iris Aperture Mechanism (Novelty: 9)

*   **Short Description:** A photorealistic or stylized camera iris mechanism that opens/closes to represent brightness.
*   **Main Screen Layout:** Overlapping iris blades.
*   **Page Structure:** 2-page.
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Smoothly opens or closes the iris blades. Wide open = 100% brightness.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe for presets.
*   **LED Ring:** Amber when on.
*   **Pros:** Very satisfying mechanical metaphor for light control.
*   **Cons:** Requires smooth animation to look good.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium (using image sequences).
*   **Include in Visual Batch:** Yes.

## Concept 18: Radar Sweep Animation (Novelty: 9)

*   **Short Description:** A rotating line reveals the current state as it sweeps around, acting as both an idle animation and an interactive control.
*   **Main Screen Layout:** Dark screen with a sweeping radar line.
*   **Page Structure:** 2-page.
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** The sweeping line follows the rotation of the knob, leaving a "trail" that represents the brightness level.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe for presets.
*   **LED Ring:** Amber when on.
*   **Pros:** Dynamic and engaging.
*   **Cons:** Constant motion might be distracting in a bedroom if not dimmed properly.
*   **Risk on 240x240:** Low.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 19: Vinyl DJ Crossfader (Novelty: 9)

*   **Short Description:** The left and right hemispheres of the screen display different light scenes, and the knob acts as a crossfader between them.
*   **Main Screen Layout:** Split screen with a central draggable thumb.
*   **Page Structure:** 1-page.
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Blends between the two scenes (e.g., Warm White on left, Nightlight on right).
*   **Knob Press:** Toggles power.
*   **Touch:** Tap hemispheres to change the assigned scene.
*   **LED Ring:** Amber when on.
*   **Pros:** Excellent for mood blending rather than just dimming.
*   **Cons:** More complex to set up initially.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium.
*   **Include in Visual Batch:** Yes.

## Concept 20: Eclipse Corona (Novelty: 9)

*   **Short Description:** A solar eclipse visualization where brightness controls how much corona is visible around a central dark moon.
*   **Main Screen Layout:** Central black circle surrounded by a glowing, dynamic corona.
*   **Page Structure:** 2-page.
*   **Best Interaction Model:** Knob-first.
*   **Knob Rotation:** Increases the intensity and spread of the corona effect.
*   **Knob Press:** Toggles power.
*   **Touch:** Swipe for presets.
*   **LED Ring:** Amber when on.
*   **Pros:** Beautiful, natural metaphor for light. Keeps the center of the screen dark (good for OLED/IPS at night).
*   **Cons:** Requires good rendering to look organic.
*   **Risk on 240x240:** Medium.
*   **ESPHome/LVGL Feasibility:** Medium (using radial gradients or image sequences).
*   **Include in Visual Batch:** Yes.
