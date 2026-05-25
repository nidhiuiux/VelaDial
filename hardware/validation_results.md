# VelaDial — Hardware Validation Results

**Status:** Active validation log — fill in incrementally as physical validation
phases execute.  
**Project:** VelaDial  
**Scope:** Single consolidated results document, as defined by
`docs/13_Firmware_Prep_Validation_Plan.md` §7.

---

## How to use this document

- One section per validation phase (per
  `docs/13_Firmware_Prep_Validation_Plan.md` §3).
- Each section is owned by a named tester for a specific board / device in
  hand.
- Do **not** invent results. Any field whose value has not been physically
  observed must be recorded as
  `NOT TESTED — awaiting Hardik physical validation input`.
- Do **not** edit `hardware/elecrow_pinout.md` based on guesses. Only update
  the pinout doc when physical deltas have been confirmed by Hardik on the
  actual board in hand.

---

## Step 15B — ELECROW Door-Side Board / Pinout Validation Results

Implements the validation defined in
`docs/validation/step_15a_elecrow_pinout_validation.md` (the Step 15A plan).

### Identification

| Field | Value |
| --- | --- |
| Board (model) | ELECROW CrowPanel 1.28 in ESP32-S3 round rotary touch display |
| Board revision / silkscreen / sticker | NOT TESTED — awaiting Hardik physical validation input |
| Validation date | NOT TESTED — awaiting Hardik physical validation input |
| Tester name | NOT TESTED — awaiting Hardik physical validation input |
| Bring-up YAML used | `esphome/door_side_rotary.yaml` (existing bring-up, unchanged) |
| Pinout source-of-truth used | `hardware/elecrow_pinout.md` |

### Evidence checklist

| Evidence item | Status | Location / reference |
| --- | --- | --- |
| Board front photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Board back photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Revision marking (silkscreen / sticker) photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Serial logs from bring-up flash captured | NOT TESTED — awaiting Hardik physical validation input | — |
| I²C scan output captured | NOT TESTED — awaiting Hardik physical validation input | — |

> Photos and logs will be attached to the Step 15B PR (or stored in an
> agreed-upon location) once Hardik provides them. Do not commit raw binary
> assets to the repo unless the team explicitly decides to.

### Subsystem results

Statuses: **PASS** / **PARTIAL** / **FAIL** / **BLOCKED** / **NOT TESTED**.
(**PARTIAL** is only acceptable with a documented workaround approved by
Hardik — see the Step 15B gate below.)

| # | Check | Expected result | Actual result | Status | Evidence / notes |
| -: | --- | --- | --- | --- | --- |
| 1 | Power-on baseline | Board powers up on USB 5 V / 2 A; power LED on GPIO40 behaves as documented; no brownout, no overheat, no reboot loop. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 2 | Compile `esphome/door_side_rotary.yaml` | ESPHome compile completes without errors for the existing bring-up YAML, no pin / component conflicts. | — | NOT TESTED — awaiting Hardik physical validation input | Compile only — no behavior changes to YAML. |
| 3 | Flash bring-up YAML | Flash succeeds; device boots; serial log shows clean boot, no panic, no strap-pin warning, no reboot loop. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 4 | Display — GC9A01A | Display initializes over SPI on the documented pins (SCLK 10, MOSI 11, DC 3, CS 9, RST 14); LVGL bring-up page renders ("VelaDial" / "Press dial"). | — | NOT TESTED — awaiting Hardik physical validation input | Confirm `invert_colors: true` is still correct for this panel; record if not. |
| 5 | Backlight PWM | Backlight on GPIO46 sweeps smoothly 0 % → 100 % via the `back_light` light entity; visible change; no flicker / whine. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 6 | CST816 I²C address `0x15` | I²C scan on SDA GPIO6 / SCL GPIO7 reports a device at `0x15`. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 7 | CST816 coordinate orientation | Touches generate coordinates; existing transform (`mirror_y: true`, `swap_xy: true`) maps touches to visible panel correctly. | — | NOT TESTED — awaiting Hardik physical validation input | Record any required transform change — do not "fix" in production YAML in Step 15B. |
| 8 | Rotary encoder CW / CCW | Rotating CW and CCW changes encoder count in opposite directions on pins A=GPIO45, B=GPIO42. | — | NOT TESTED — awaiting Hardik physical validation input | Record observed direction. Do not "fix" direction in production YAML in Step 15B. |
| 9 | Rotary encoder press | Pressing the encoder shaft registers a press on GPIO41 (`rotary_button` in bring-up YAML). | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 10 | WS2812 LED ring (5 LEDs) | All 5 LEDs on GPIO48 light up in order from a temporary test entity; no dropped pixels. | — | NOT TESTED — awaiting Hardik physical validation input | Use a temporary test entity only — do not commit it to production YAML. |
| 11 | 30-minute stability soak | Board runs ≥ 30 min with display + backlight + touch active; no reboots, no I²C bus errors, no thermal issues. | — | NOT TESTED — awaiting Hardik physical validation input | Attach final serial-log tail to evidence. |

### Pinout delta vs `hardware/elecrow_pinout.md`

#### Confirmed matches

| Subsystem | Function | Documented (`hardware/elecrow_pinout.md`) | Physically confirmed? |
| --- | --- | --- | --- |
| Display | SCLK | GPIO10 | NOT TESTED — awaiting Hardik physical validation input |
| Display | MOSI | GPIO11 | NOT TESTED — awaiting Hardik physical validation input |
| Display | DC | GPIO3 | NOT TESTED — awaiting Hardik physical validation input |
| Display | CS | GPIO9 | NOT TESTED — awaiting Hardik physical validation input |
| Display | RST | GPIO14 | NOT TESTED — awaiting Hardik physical validation input |
| Display | Backlight (PWM) | GPIO46 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | SDA | GPIO6 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | SCL | GPIO7 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | INT | GPIO5 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | RST | GPIO13 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | I²C address `0x15` | `0x15` | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | A | GPIO45 | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | B | GPIO42 | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | Switch | GPIO41 | NOT TESTED — awaiting Hardik physical validation input |
| WS2812 | Data | GPIO48 | NOT TESTED — awaiting Hardik physical validation input |
| WS2812 | LED count | 5 | NOT TESTED — awaiting Hardik physical validation input |
| Power indicator | Power LED | GPIO40 | NOT TESTED — awaiting Hardik physical validation input |

#### Confirmed mismatches

| Subsystem | Function | Documented | Observed on board | Notes / proposed action |
| --- | --- | --- | --- | --- |
| _none yet_ | — | — | — | NOT TESTED — awaiting Hardik physical validation input |

> ⚠️ Do **not** edit `hardware/elecrow_pinout.md` based on entries here until
> Hardik has physically confirmed the delta and explicitly approved the
> pinout-doc update in a separate, narrowly-scoped PR.

#### Unresolved TBDs

| Item | Why it is TBD | Resolution path |
| --- | --- | --- |
| Board revision / silkscreen | Not yet observed in hand | Photograph silkscreen, record value here. |
| Onboard OLED presence (`SDA GPIO38`, `SCL GPIO39`) | Optional on some board revisions | Visual inspection + I²C scan on bus, record presence/absence. |
| Test IO usage (`GPIO4`, `GPIO12`) | Listed as test IO in pinout doc; intended door-side function not finalized | Out of scope for Step 15B unless they conflict with bring-up. |

---

## Step 15B Gate

> 🛑 **Do not proceed to production door-side firmware, production door-side
> YAML, the 3-page LVGL UI, door-side sensor wiring (TSL2591 / SHT45), Home
> Assistant action wiring, VL53L4CD work, or any sensor fusion work until
> Step 15B is completed end-to-end and the results in this document have been
> reviewed and approved by Hardik.**

Specifically, until this gate is lifted, do **not**:

- Modify `esphome/door_side_rotary.yaml` for production behavior.
- Create any new production YAML under `esphome/`.
- Begin door-side sensor (TSL2591, SHT45) validation
  (`docs/13_Firmware_Prep_Validation_Plan.md` §3.C).
- Begin VL53L4CD / VL53L0X work
  (`docs/13_Firmware_Prep_Validation_Plan.md` §3.A / §3.E).
- Begin LVGL Power / Brightness / Presets page implementation.
- Begin sensor fusion work (explicitly out of v1).

Gate lift requires: all subsystem rows above marked **PASS** (or **PARTIAL**
with documented workaround approved by Hardik), all evidence items captured,
and any pinout deltas resolved.

---

## Phase 1 — Door-side ESP32-S3 Firmware Draft
	
	**Date:** 2026-05-24  
	**Status:** Draft created (PR #19 merged)  
	**Compile status:** PASSED in Manus environment using ESPHome 2026.5.0 (binary size 1,194,131 bytes, 0 errors, 4 non-blocking warnings). Flash and physical validation remain NOT TESTED.  
	
	A draft production-oriented YAML (`esphome/door_side_rotary.yaml`) has been created. The YAML compiled without errors in the sandbox environment (ESPHome 2026.5.0, ESP-IDF framework). It has **not** been flashed to or tested on physical hardware. The Step 15B gate above remains **unlifted** — physical board validation is still required before this draft can be considered production-ready.
	
	## Phase 2 — Bedside ESP32-C6 APDS Firmware Draft
	
	**Date:** 2026-05-24  
	**Status:** Draft created  
	**Compile status:** PASSED (ESPHome 2026.5.0, ESP-IDF, 0 errors, 0 warnings, 181.5s)  
	**Physical Validation:** NOT TESTED  
	
	A draft production-oriented YAML (`esphome/bedside_gesture.yaml`) has been created on branch `firmware/bedside-apds-v1-draft`. It uses native ESPHome APDS-9960 gesture support. No sensor fusion is implemented. Physical validation remains pending.

### Pin Cross-Validation (Source Research)

The YAML pin assignments were cross-validated against **three independent sources** on 2026-05-24:

| Source | URL | Agreement |
| --- | --- | --- |
| Elecrow Official GitHub (factory source code) | https://github.com/Elecrow-RD/CrowPanel-1.28inch-HMI-ESP32-Rotary-Display-240-240-IPS-Round-Touch-Knob-Screen | Full match |
| Incipiens Community ESPHome YAML (XDA/HA) | https://github.com/Incipiens/Elecrow-Rotary-Displays | Full match |
| Makerguides.com Tutorial (pin table) | https://www.makerguides.com/getting-started-crowpanel-1-28inch-hmi-esp32-rotary-display/ | Full match |

All three sources confirm: Display SPI (SCLK=10, MOSI=11, CS=9, DC=3, RST=14), Backlight=GPIO46, Touch (SDA=6, SCL=7, INT=5, RST=13, addr 0x15), Encoder (A=45, B=42, SW=41), WS2812 (GPIO48, 5 LEDs), Power LED (GPIO40), invert_colors=true, touch transform (mirror_y=true, swap_xy=true).

This does NOT replace physical validation. It confirms the YAML is aligned with all known documentation sources.

---

## Phase 3 — Raspberry Pi / Home Assistant Setup Guide

**Date:** 2026-05-25  
**Status:** Setup guide created (`docs/setup/raspberry_pi_home_assistant_setup.md`)  
**HA / ESPHome / LocalTuya Setup:** NOT TESTED  
**Physical PASS Results Added:** No  

A comprehensive setup guide has been created documenting the intended deployment path for Home Assistant OS on Raspberry Pi, ESPHome add-on configuration, LocalTuya integration, and `light.bedroom_group` creation. All steps remain NOT TESTED until Hardik performs the physical setup.

---

## Phase 4 — Full E2E Setup and Validation Guide

**Date:** 2026-05-25  
**Status:** E2E guide created (`docs/setup/full_e2e_setup_and_validation_guide.md`)  
**E2E Validation:** NOT TESTED  
**Physical PASS Results Added:** No  

A full E2E setup and validation guide has been created. It defines the complete validation sequence for door-side, bedside, HA/LocalTuya command path, and full system scenarios. All results remain NOT TESTED until Hardik performs them on physical hardware.

---

## Phase 5 — UI/UX Guide Integration Plan

**Date:** 2026-05-25  
**Status:** UI/UX integration plan created (`docs/ui/ux_guide_integration_plan.md`)  
**UI Implementation:** NOT PERFORMED  
**Hardik UI/UX Guide:** PENDING HARDIK INPUT  
**Physical PASS Results Added:** No  

A UI/UX guide integration plan has been created documenting the agreed visual themes, locked v1 rules, pending inputs from Hardik, integration decision matrix, and acceptance criteria for future UI implementation. No UI code changes were made. No ESPHome YAML was edited. Physical validation remains NOT TESTED.

---

## Phase 6 — Claude Review Package

**Date:** 2026-05-25  
**Status:** Claude review package created (`docs/review/claude_review_package.md`)  
**Validation Performed:** No  
**Physical PASS Results Added:** No  

A Claude review package has been created for independent review of the VelaDial repository after Phases 0–5. This is a documentation-only deliverable. No validation was performed, no firmware was changed, and all hardware/E2E validation remains NOT TESTED.

---

## Document control

**Version:** 0.6 — Added Phase 6 Claude review package note; no validation performed.  
**Owner approval required:** Yes, before lifting the Step 15B gate.  
**Next phase after sign-off:** Door-side sensor validation
(`docs/13_Firmware_Prep_Validation_Plan.md` §3.C).
