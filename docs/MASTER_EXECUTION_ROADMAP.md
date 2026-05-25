# Master Execution Roadmap

**Project:** VelaDial  
**Status:** Active  
**Purpose:** This document serves as the real execution contract for the VelaDial project. It maps out the entire remaining build sequence, defining all validation gates, decision points, phase dependencies, and strict definitions of done.

---

## SECTION A — Current Project State

The project has established a solid documentation foundation, but firmware and hardware validation remain in preliminary states. Pull requests #1 through #17 have been successfully merged. Notably, PR #16 added the `hardware/validation_results.md` file, and PR #17 added the `docs/validation/elecrow_source_confirmation_matrix.md` file.

While the ELECROW GPIOs are source-confirmed, physical hardware validation remains strictly **NOT TESTED**. The existing YAML files (`esphome/door_side_rotary.yaml` and `esphome/bedside_gesture.yaml`) are bring-up/starter configurations only. No production firmware has been validated, and no hardware PASS results currently exist.

---

## SECTION B — Locked v1 Scope

The following behaviors and constraints are strictly locked for the version 1 (v1) build.

### Door-side Controller
The door-side UI consists of exactly three pages: Power, Brightness, and Presets. There is no required Environment page, and the SHT45 sensor is reserved for secondary or future diagnostics only. The Presets page features exactly four locked presets: Warm White, Soft Amber, Neutral White, and Low Nightlight.

The device employs a strict wake-only-first behavior. Touching the screen, rotating the knob, or pressing the knob while the device is asleep will only wake the display. A second deliberate action while the display is awake is required to send a command or toggle a state.

Page navigation is handled via horizontal swipes: a left swipe moves to the next page, and a right swipe moves to the previous page. A 3-dot indicator displays the current page, with the active dot colored amber.

Knob press behavior varies by page:
- **Power page:** Toggles the bedroom light group.
- **Brightness page:** Returns to the Power page.
- **Presets page:** Applies the currently highlighted preset.
- **While asleep:** Wakes the display only.

### Bedside Controller
The bedside controller utilizes APDS-9960 standalone gestures for v1. A left gesture turns the bedroom group off, while a right gesture turns the bedroom group on. A cooldown period prevents repeated triggers.

The VL53L4CD standalone hand-hold nightlight is included in v1 only if its support path is verified and implemented. Crucially, VL53L4CD and APDS-9960 sensor fusion is reserved for v2 or future builds; there is absolutely no sensor fusion in v1.

### System Path
The control path flows from the ESPHome devices to Home Assistant running on a Raspberry Pi, then via LocalTuya over the local LAN to the `light.bedroom_group`.

---

## SECTION C — Source Confirmed vs Physical Test Pending

**⚠️ WARNING: Source-confirmed does not equal physically validated.**

| Item | Source-Confirmed Status | Physical Status | Evidence File | Next Action |
| :--- | :--- | :--- | :--- | :--- |
| ELECROW GPIOs | Confirmed | NOT TESTED | `docs/validation/elecrow_source_confirmation_matrix.md` | Awaiting Hardik physical validation |
| Physical board revision | Pending | NOT TESTED | `hardware/validation_results.md` | Photograph silkscreen/sticker |
| ESPHome compile | Pending | NOT TESTED | `hardware/validation_results.md` | Run compile command |
| Flash | Pending | NOT TESTED | `hardware/validation_results.md` | Flash device and capture logs |
| Serial logs | Pending | NOT TESTED | `hardware/validation_results.md` | Capture clean boot logs |
| I2C scan | Pending | NOT TESTED | `hardware/validation_results.md` | Run I2C scan on both buses |
| Display test | Pending | NOT TESTED | `hardware/validation_results.md` | Verify GC9A01A output |
| Touch test | Pending | NOT TESTED | `hardware/validation_results.md` | Verify CST816 coordinates |
| Encoder test | Pending | NOT TESTED | `hardware/validation_results.md` | Verify CW/CCW and press |
| WS2812 test | Pending | NOT TESTED | `hardware/validation_results.md` | Verify 5 LEDs |
| 30-minute soak | Pending | NOT TESTED | `hardware/validation_results.md` | Run stability test |

---

## SECTION D — Firmware Execution Plan

This section outlines the plan for firmware development across the three target platforms. No firmware is written in this phase.

### Target 1: Door-side ELECROW ESP32-S3
- **Intended YAML file path:** `esphome/door_side_rotary.yaml`
- **Intended entities:** Display, touch, rotary encoder, backlight PWM, WS2812 LEDs, TSL2591, SHT45.
- **Display/LVGL features:** 3-page UI (Power, Brightness, Presets), 3-dot indicator, amber accents.
- **Wake-only-first behavior plan:** Implement logic to intercept first input when asleep.
- **Power/brightness/preset behavior:** Map knob and touch inputs to specific HA actions per page.
- **Backlight behavior:** TSL2591 lux drives PWM locally.
- **Secrets handling:** Use `!secret` for WiFi, API, and OTA passwords.
- **Compile command:** `esphome compile esphome/door_side_rotary.yaml`
- **Flash command:** `esphome run esphome/door_side_rotary.yaml`
- **Validation checklist:** Display renders, touch accurate, encoder responsive, backlight adapts.
- **Rollback plan:** Revert to bring-up YAML.
- **Untested items:** All physical hardware interactions.

### Target 2: Bedside ESP32-C6 Feather
- **Intended YAML file path:** `esphome/bedside_gesture.yaml`
- **APDS-9960 gesture logic:** Standalone left/right detection.
- **Left/right actions:** Left = off, Right = on.
- **Cooldown behavior:** ~1s delay after successful gesture.
- **Home Assistant target entity:** `light.bedroom_group`
- **Secrets handling:** Use `!secret` for WiFi, API, and OTA passwords.
- **Compile command:** `esphome compile esphome/bedside_gesture.yaml`
- **Flash command:** `esphome run esphome/bedside_gesture.yaml`
- **Validation checklist:** Gestures trigger HA actions, cooldown prevents spam.
- **Rollback plan:** Revert to bring-up YAML.
- **Untested items:** All physical hardware interactions.
- **Explicitly no sensor fusion:** APDS-9960 operates entirely independently of VL53L4CD.

### Target 3: Raspberry Pi / Home Assistant
- **Home Assistant role:** Central hub for state management and automation.
- **ESPHome add-on role:** Compiles and manages ESP32 nodes.
- **LocalTuya role:** Provides local LAN control of Tuya bulbs.
- **`light.bedroom_group`:** The single target entity for all lighting commands.
- **Network assumptions:** Local LAN must be stable; internet not required for normal control.
- **Static IP/reservation recommendation:** Assign static IPs to Pi, ESP32s, and Tuya bulbs.
- **secrets.yaml handling:** Store all sensitive keys locally on the Pi.
- **Backup/restore:** Configure automated HA backups.
- **Troubleshooting/log collection:** Use HA log viewer and ESPHome serial logs.

---

## SECTION E — UI/UX Guide Integration Plan

### Pending Hardik UI/UX Guide Integration
Hardik possesses UI/UX guidance that may not yet be present in the GitHub repository.

- **What Hardik must provide:** The detailed UI/UX guide or specific design assets.
- **How it will be integrated:** The guide will be reviewed and its styling/animation directives applied to the LVGL implementation.
- **Rule:** The UI guide cannot add pages unless Hardik explicitly approves.
- **Locked v1:** The interface remains exactly 3 pages.
- **No Environment page:** This remains excluded unless approved for a future v2.
- **Premium UI:** Premium UI directives can affect styling and animation, but cannot alter the core v1 scope unless explicitly approved.

---

## SECTION F — Corrected Phase Plan

The execution must strictly follow this sequence.

| Phase | Output | Details |
| :--- | :--- | :--- |
| **Phase 0 — Master Execution Roadmap** | `docs/MASTER_EXECUTION_ROADMAP.md` | Current task. PR required. |
| **Phase 1 — Door-side ESP32-S3 firmware draft** | Draft update to `esphome/door_side_rotary.yaml` | **IN REVIEW** — draft created; compile reported successful; physical validation pending; PR pending. |
| **Phase 2 — Bedside ESP32-C6 APDS firmware draft** | Draft update to `esphome/bedside_gesture.yaml` | APDS left/right gestures, cooldown, no sensor fusion, no VL53L4CD unless separately approved. |
| **Phase 3 — Raspberry Pi / Home Assistant setup guide** | `docs/setup/raspberry_pi_home_assistant_setup.md` | Document HA, ESPHome, and LocalTuya configuration. |
| **Phase 4 — Full E2E setup and validation guide** | `docs/setup/full_e2e_setup_and_validation_guide.md` | Comprehensive testing plan. |
| **Phase 5 — UI/UX guide integration plan** | `docs/ui/ux_guide_integration_plan.md` | Integrate Hardik-provided UI guide if available. |
| **Phase 6 — Claude review package** | `docs/review/claude_review_package.md` | Prepare documentation for external review. |
| **Phase 7 — Optional VL53L4CD decision/implementation path** | Implementation depends on decision | Only after Hardik chooses: custom component, defer ToF nightlight, or VL53L0X fallback. Do not make VL53L4CD block all other work. |

---

## SECTION G — Hardware E2E Test Backlog

The following tests must be completed and recorded as PASS before the system is considered validated.

- ELECROW board revision photo
- Front/back board photos
- Compile result
- Flash result
- Serial boot logs
- I2C scan
- GC9A01A display test
- Backlight PWM test
- CST816 touch address test
- CST816 touch coordinate/orientation test
- Rotary CW/CCW test
- Rotary press test
- WS2812 5 LED test
- 30-minute stability soak
- ESP32-C6 compile
- ESP32-C6 flash
- APDS-9960 I2C scan
- APDS left gesture test
- APDS right gesture test
- Gesture cooldown test
- Home Assistant service call test
- LocalTuya command path test
- `light.bedroom_group` on/off test
- Brightness test
- Preset test
- VL53L4CD support-gated test (only if approved)

---

## SECTION H — Objective Completion Matrix

| Objective | Status | Evidence | File / PR | Next Action |
| :--- | :--- | :--- | :--- | :--- |
| Roadmap | DONE | This document | `docs/MASTER_EXECUTION_ROADMAP.md` | PR #18 merged |
| Door firmware draft | DONE | PR #19 merged | `esphome/door_side_rotary.yaml` | compile PASSED; physical validation pending |
| Bedside firmware draft | DONE | PR #20 merged | `esphome/bedside_gesture.yaml` | compile PASSED; physical validation pending |
| Raspberry Pi guide | DONE | PR #21 merged | `docs/setup/raspberry_pi_home_assistant_setup.md` | HA/ESPHome/LocalTuya setup NOT TESTED |
| E2E guide | IN REVIEW | None | `docs/setup/full_e2e_setup_and_validation_guide.md` | guide created; E2E validation NOT TESTED |
| UI/UX guide integration | NOT STARTED | None | `docs/ui/ux_guide_integration_plan.md` | Start Phase 5 |
| Claude review package | NOT STARTED | None | `docs/review/claude_review_package.md` | Start Phase 6 |
| ELECROW physical validation | HARDWARE TEST PENDING | None | `hardware/validation_results.md` | Hardik to test |
| Bedside APDS validation | HARDWARE TEST PENDING | None | `hardware/validation_results.md` | Hardik to test |
| HA command-path validation | HARDWARE TEST PENDING | None | `hardware/validation_results.md` | Hardik to test |
| VL53L4CD decision | BLOCKED | None | `docs/vl53l4cd_support_verification.md` | Hardik to decide |
| No sensor fusion compliance | IN PROGRESS | Roadmap constraints | Multiple | Maintain constraint |
| Secrets safety | IN PROGRESS | Roadmap constraints | Multiple | Maintain constraint |

---

## SECTION I — Definition of Done

The project is not considered "done" until all of the following conditions are met:

- The roadmap exists and is approved.
- The door firmware draft exists.
- The bedside firmware draft exists.
- The Raspberry Pi/Home Assistant setup guide exists.
- The full E2E setup/validation guide exists.
- The UI/UX guide is integrated or clearly marked as pending.
- The Claude review package exists.
- All hardware tests are listed in the backlog.
- All untested items are explicitly marked as NOT TESTED or HARDWARE TEST PENDING.
- No sensor fusion has been added to the v1 implementation.
- No unsupported VL53L4CD claims are made.
- No secrets or credentials are committed to the repository.
- All branches are pushed to the remote repository.
- All Pull Requests are created.
- Hardik manually reviews and merges all Pull Requests.

---

## SECTION J — GitHub / PR Requirement

- If GitHub access works: Push the branch and open a Pull Request.
- If GitHub access does not work: Mark GitHub PR creation as BLOCKED and provide the exact error.
- Do not substitute a zip file for a PR without explicitly marking PR creation as BLOCKED.
