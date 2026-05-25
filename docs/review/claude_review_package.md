# VelaDial: Claude Review Package

## 1. Review Purpose

This package is designed for Claude to independently review the VelaDial repository after the completion of Phases 0 through 5. The purpose of this review is to ensure that the firmware drafts, setup guides, and integration plans adhere strictly to the locked v1 scope, maintain safety and security standards, and correctly identify all pending hardware validation steps. This package is for independent review only, not for implementation.

## 2. Current Merged Phase Summary

The following pull requests have been merged into the `main` branch, representing the completion of Phases 0 through 5:

| PR | Title | Files Changed | Purpose | Known Status | What Remains Untested |
| :--- | :--- | :--- | :--- | :--- | :--- |
| #18 | Master execution roadmap | `docs/MASTER_EXECUTION_ROADMAP.md` | Establish the 6-phase build sequence and validation gates. | Merged | N/A (Documentation) |
| #19 | Door-side ESP32-S3 firmware draft | `esphome/door_side_rotary.yaml`, `hardware/validation_results.md`, `docs/MASTER_EXECUTION_ROADMAP.md` | Draft production YAML for the door-side controller. | Compile PASSED | Physical hardware validation |
| #20 | Bedside ESP32-C6 APDS firmware draft | `esphome/bedside_gesture.yaml`, `hardware/validation_results.md`, `docs/MASTER_EXECUTION_ROADMAP.md`, `docs/REFERENCE_RESOURCES.md` | Draft production YAML for the bedside gesture controller. | Compile PASSED | Physical hardware validation |
| #21 | Raspberry Pi / Home Assistant setup guide | `docs/setup/raspberry_pi_home_assistant_setup.md`, `docs/MASTER_EXECUTION_ROADMAP.md`, `docs/REFERENCE_RESOURCES.md`, `hardware/validation_results.md` | Document HA OS, ESPHome, and LocalTuya setup. | Guide created | HA/ESPHome/LocalTuya setup |
| #22 | Full E2E setup and validation guide | `docs/setup/full_e2e_setup_and_validation_guide.md`, `docs/MASTER_EXECUTION_ROADMAP.md`, `hardware/validation_results.md` | Define the complete E2E validation sequence. | Guide created | All E2E validation scenarios |
| #23 | UI/UX guide integration plan | `docs/ui/ux_guide_integration_plan.md`, `docs/MASTER_EXECUTION_ROADMAP.md`, `docs/REFERENCE_RESOURCES.md`, `hardware/validation_results.md` | Plan UI integration based on agreed themes. | Plan created | UI implementation |

## 3. Current System Scope

The locked v1 scope for VelaDial is strictly defined to ensure a focused and stable initial release. The door-side UI is limited to exactly three pages: Power, Brightness, and Presets. The Presets page must contain exactly four options: Warm White, Soft Amber, Neutral White, and Low Nightlight. There is no Environment page in this build, and SHT45 data is reserved for secondary or future diagnostics only.

For the bedside UI, functionality is restricted to APDS standalone gestures, where a left gesture turns the bedroom group off, and a right gesture turns it on. There is no sensor fusion in v1, and VL53L4CD support is gated and not implemented. Finally, the LocalTuya path specifically targets `light.bedroom_group`.

## 4. Firmware Review Request

Claude is requested to review the firmware files `esphome/door_side_rotary.yaml` and `esphome/bedside_gesture.yaml`. 

The review must specifically check for ESPHome and LVGL syntax risks, as well as ESP32-S3 and ESP32-C6 board compatibility. It is crucial to verify APDS-9960 support correctness and ensure GPIO assignments match the source confirmation matrix (`docs/validation/elecrow_source_confirmation_matrix.md`). The review should also confirm the strict enforcement of the wake-only-first rule, validate Home Assistant action/service syntax, and ensure safe secrets handling with no hardcoded credentials. Furthermore, Claude must verify the clear separation between compile status reporting and physical validation, confirm the absence of sensor fusion and VL53L4CD implementation, and ensure there are no accidental dependencies on unavailable hardware.

## 5. Door-Side Review Checklist

For the door-side firmware, Claude must verify that exactly three pages exist and exactly four presets are defined, with no Environment page present. The review must confirm that the wake-only-first guard is applied across all input paths, including properly guarded page swipes. Additionally, knob press behavior must be correctly defined per page, and brightness arc direct touch must be explicitly marked as implementation pending. Finally, Claude should ensure that TSL2591/SHT45 data is not promoted to the main UI and that the physical validation pending status remains clear.

## 6. Bedside Review Checklist

For the bedside firmware, Claude must verify that the APDS native gesture syntax is correct, with the LEFT gesture mapping to off and the RIGHT gesture mapping to on. The review should confirm that cooldown logic is implemented and that GPIO20 STEMMA QT enable logic is present. Furthermore, the I2C pins must match the Adafruit ESP32-C6 Feather specification. Claude must also ensure that no VL53L4CD or sensor fusion logic is present, and that the proximity diagnostic risk is noted.

## 7. Setup Docs Review Checklist

Claude must review the setup documentation, specifically `docs/setup/raspberry_pi_home_assistant_setup.md` and `docs/setup/full_e2e_setup_and_validation_guide.md`. This review should evaluate the accuracy of the Raspberry Pi guide and the correctness of the ESPHome setup flow. It must also verify `secrets.yaml` placeholder safety, `light.bedroom_group` creation instructions, and LocalTuya caution and key handling guidance. Finally, Claude should check the backup/recovery instructions, E2E validation checklist completeness, evidence collection guidance, and failure triage completeness.

## 8. UI/UX Review Checklist

Claude must review the UI/UX integration plan (`docs/ui/ux_guide_integration_plan.md`) to ensure the preservation of the v1 scope and confirm that no new UI pages, such as an Environment page, are added. The review should verify a clear indication that the Hardik UI/UX guide is pending and check for an accurate listing of implementation constraints. Additionally, Claude must confirm that wake-only-first preservation is maintained by design and evaluate the acceptance criteria for future UI changes.

## 9. Validation Status and Risk Summary

| Area | Current Status | Evidence | Risk | Claude Review Request |
| :--- | :--- | :--- | :--- | :--- |
| Door firmware | Compile reported PASS, hardware NOT TESTED | PR #19 | Pin/orientation mismatch | Verify GPIOs and wake guards |
| Bedside firmware | Compile reported PASS, hardware NOT TESTED | PR #20 | I2C/gesture reliability | Verify APDS syntax and GPIO20 |
| HA setup guide | Created, setup NOT TESTED | PR #21 | Network/OS issues | Verify HA/ESPHome flow |
| LocalTuya path | Documented, NOT TESTED | PR #21 | DP mapping errors | Verify Tuya caution notes |
| E2E guide | Created, NOT TESTED | PR #22 | Incomplete scenarios | Verify validation completeness |
| UI/UX plan | Created, implementation pending | PR #23 | Scope creep | Verify v1 constraints |
| VL53L4CD | Decision BLOCKED | `docs/vl53l4cd_support_verification.md` | Hardware unsupported | Confirm absence in YAML |
| Brightness arc | IMPLEMENTATION PENDING | `esphome/door_side_rotary.yaml` | LVGL 9 opaque struct | Confirm documented limitation |

## 10. Known Blockers / Pending Hardik Input

Several items remain blocked or pending input from Hardik. These include physical ELECROW validation (Step 15A/15B) and ESP32-C6 + APDS physical validation. Execution of the HA/LocalTuya setup and the creation and verification of `light.bedroom_group` are also pending. Furthermore, hardware photos and logs, along with the Hardik UI/UX guide and assets, have not yet been provided. Finally, a decision regarding the VL53L4CD—whether to use a custom component, defer ToF, or use a VL53L0X fallback with approval—remains blocked.

## 11. Exact Claude Prompt

Copy and paste the following prompt into Claude to initiate the review:

```text
Please perform an independent review of the VelaDial repository (Phases 0-5). 

Provide your review in the following format:
1. PASS / REQUEST CHANGES (Overall verdict)
2. Blocking issues (if any)
3. Non-blocking notes
4. File-by-file review
5. YAML syntax risk review
6. Hardware assumption risk review
7. Security/secrets review
8. Scope compliance review (ensure strict adherence to v1 locked scope)
9. Recommended next PRs

Focus specifically on:
- esphome/door_side_rotary.yaml (check wake-only-first guards, 3 pages, 4 presets, GPIOs)
- esphome/bedside_gesture.yaml (check APDS syntax, GPIO20, no VL53L4CD)
- docs/setup/raspberry_pi_home_assistant_setup.md (check secrets safety, LocalTuya flow)
- docs/setup/full_e2e_setup_and_validation_guide.md (check validation completeness)
- docs/ui/ux_guide_integration_plan.md (check v1 scope preservation)

Confirm that no physical PASS claims are made and that all hardware/E2E validation is clearly marked as NOT TESTED.
```

## 12. Explicit Non-Goals

This phase and its associated pull request explicitly exclude any code changes or firmware edits. No physical PASS claims are made, and there is no inclusion of sensor fusion logic or VL53L4CD implementation. Furthermore, absolutely no secrets, logs, or binaries are committed to the repository.
