# VelaDial: UI/UX Guide Integration Plan

## 1. Scope and Status

This document serves as the **planning and integration guide** for the VelaDial door-side display UI. 

**Important Status Notes:**
- **This is a planning document only.** No UI implementation code is included in this phase.
- **No ESPHome YAML is changed** by this document.
- **Physical validation remains NOT TESTED.**
- The detailed UI/UX guide from Hardik is **PENDING HARDIK INPUT** (unless explicitly provided in future iterations). The themes and rules below are based on the agreed-upon `docs/04_UI_UX_Design_Brief.md`.

## 2. Locked v1 UI Rules

The following rules are locked for the v1 firmware and must not be violated by any UI design or animation. The interface is strictly limited to exactly three pages: Power, Brightness, and Presets. The Presets page must contain exactly four options: Warm White, Soft Amber, Neutral White, and Low Nightlight. There is no Environment page in the first build, nor is there any sensor fusion or VL53L4CD UI unless the support path is separately approved.

Furthermore, SHT45 data is reserved for secondary or future diagnostics and does not belong on the main three pages. The brightness arc direct touch functionality is currently marked as **IMPLEMENTATION PENDING** due to LVGL 9 constraints in ESPHome, making the physical knob the primary brightness input. Finally, the wake-only-first rule must be strictly preserved; animations, touch effects, or visual feedback must never accidentally trigger a room light command when waking the display from sleep.

## 3. Existing UI State

The current state of the door-side UI (`esphome/door_side_rotary.yaml`) includes a draft LVGL UI that implements the three locked pages. The YAML compilation has been reported as **PASSED**, but hardware validation remains **PENDING**. Physical display rendering, touch orientation, and encoder behavior are currently **NOT TESTED**. Additionally, direct arc drag is explicitly marked as implementation pending.

## 4. Agreed UI Themes and Visual Specs

Based on `docs/04_UI_UX_Design_Brief.md`, the visual themes are agreed upon and must guide the final implementation. The interface will be dark-mode only, utilizing a pure black background (`#000000`) to seamlessly blend with the physical bezel. A warm amber accent (`#FFB300`) will highlight active states and brightness levels, while white text (`#FFFFFF`) ensures high legibility in dark rooms.

Typography will rely on a large, readable sans-serif font, such as Roboto, avoiding long paragraph labels. The geometry of the UI is optimized for a simple round screen, ensuring that all touch targets remain within the inscribed square to prevent cut-off corners. Motion will be kept to a minimum, avoiding aggressive animations or bright full-screen flashes. While the UI layout remains static, ambient awareness is achieved through the TSL2591 sensor, which dynamically drives the backlight intensity across Bright, Dim, and Night modes.

## 5. Pending Hardik UI/UX Input

To proceed with actual UI code implementation, specific inputs are required from Hardik. These include a visual style guide with exact hex codes if they differ from the brief, and specific typography files that must be compile-safe `.ttf` or `.woff` formats. Spacing expectations, such as padding and margins for the 240x240 canvas, are also needed.

Furthermore, any animation or motion rules, including specific transition speeds or easing curves, must be defined. Exact icon files (SVG or PNG) or Material Design icon names are required, along with page wireframes detailing the exact layout positioning for the three pages. Expectations for touch behavior, such as visual feedback on press, and sleep/wake visual behavior, like dimming curves, must be clarified. Finally, preferences for backlight behavior, LED ring color and intensity, accessibility standards like minimum font sizes, and any screenshots or visual mockups should be provided.

## 6. Integration Decision Matrix

When Hardik provides the pending inputs, use this matrix to determine how to integrate them.

| UI/UX Input | Impact Area | Can apply in v1? | Requires YAML change? | Requires hardware test? | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Colors | Global theme | Yes | Yes | Yes | Must maintain high contrast. |
| Fonts | Typography | Yes | Yes | Yes | Must be compile-safe and fit in flash memory. |
| Icons | Buttons/Labels | Yes | Yes | Yes | Use ESPHome built-in Material fonts or small images. |
| Animations | Transitions | Yes (Minimal) | Yes | Yes | Heavy animations may hurt ESP32-S3 performance. |
| Page layout | All 3 pages | Yes | Yes | Yes | Must fit 240x240 round screen. |
| Brightness UI | Arc widget | Yes | Yes | Yes | Direct touch remains pending; visual styling is allowed. |
| Preset buttons | Presets page | Yes | Yes | Yes | Must remain exactly 4 presets. |
| Page indicator | Navigation | Yes | Yes | Yes | 3-dot amber/white style. |
| Sleep/wake state | Backlight/UI | Yes | Yes | Yes | Must preserve wake-only-first rule. |
| LED ring | WS2812 | Yes | Yes | Yes | Amber glow when lights on. |
| Sound/Haptics | Feedback | No | No | No | Unsupported hardware in v1. |
| Environment page | Navigation | No | No | No | Explicitly deferred to v2/future. |

## 7. Implementation Constraints

Several technical constraints limit the UI implementation. The screen real estate is restricted to a 240x240 pixel round display, meaning corners are physically cut off. All UI elements must be defined in ESPHome YAML using the `lvgl` component, avoiding complex custom C++ LVGL code. Compile-safety is paramount; all fonts, images, and styles must compile successfully within the ESPHome environment before a pull request is opened.

Additionally, LVGL 9 limitations, specifically the opaque struct issue, prevent easy reading of the arc value in lambdas, which currently blocks direct touch brightness control. Hardware validation is also required; touch transforms (`swap_xy`, `mirror_y`) and encoder direction must be physically validated before the UI can be considered final. Lastly, while the ESP32-S3 is capable, heavy overlapping animations or massive image assets can cause UI lag or memory exhaustion, necessitating a focus on performance.

## 8. Proposed v1 UI Acceptance Criteria

Before any UI implementation pull request is merged, it must meet a strict set of acceptance criteria. The Power page must be highly readable from a distance, and the Brightness page must clearly display both the percentage and the arc. The Presets page is required to show exactly four distinct, tappable targets. Navigation must be supported by a visible 3-dot indicator that correctly highlights the active page in amber, without any page drift or layout breaking during transitions.

Crucially, the wake-only-first rule must be strictly preserved, ensuring no accidental commands are sent upon waking the display. The knob press behavior must remain unchanged, functioning as Toggle, Return, or Apply depending on the context. The UI must be usable and not blinding in a dark room, while remaining legible in a bright room. Finally, no untested direct arc touch behavior should be claimed to work.

## 9. Future v2 UI Ideas

Several ideas are explicitly out of scope for v1 but are recorded for future iterations. These include an Environment Page, which would serve as a fourth page displaying SHT45 temperature and humidity data. Advanced diagnostics could also be introduced, providing a dedicated view for sensor health, IP address, and uptime.

Future updates may explore custom animations, such as complex page transitions or fluid arc physics. Additionally, a VL53L4CD UI could offer visual feedback on the door display when the bedside ToF sensor detects a hand-hold. Sensor fusion visualizations might be developed as a debug UI to show APDS and ToF data combining. Finally, advanced LED ring features, like chasing animations or ambient-aware color shifting on the WS2812 ring, could be considered.

## 10. Required Review Process Before UI Implementation

To prevent scope creep and broken builds, a strict review process must be followed for any UI code changes. The process begins with Hardik providing the UI/UX guide or specific assets. Manus then maps each requirement to either v1 (allowed) or v2 (deferred) using the Decision Matrix. Following this, ChatGPT reviews the integration plan.

Once the plan is approved, Manus implements the allowed changes in a new branch. The YAML must compile successfully locally before Manus opens a pull request. Claude then reviews the PR against the v1 rules. Finally, Hardik flashes the branch and physically tests the UI, and only Hardik merges the PR after a physical PASS is confirmed.

## 11. Explicit Non-Goals

To maintain strict project boundaries, this document and its associated pull request explicitly exclude several elements. There is no code implementation or YAML edits, nor are there any firmware behavior changes. The introduction of new UI pages, such as an Environment page, is strictly prohibited.

Furthermore, this phase does not include any sensor fusion logic or VL53L4CD implementation. No hardware PASS claims are made, and absolutely no secrets, logs, or binaries are committed to the repository.
