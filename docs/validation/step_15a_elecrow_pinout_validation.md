# Step 15A — ELECROW Door-Side Board / Pinout Validation

**Status:** Validation artifact — first real build/validation step  
**Date:** 2026-05-22  
**Project:** VelaDial  
**Phase:** Validation execution (post-audit, pre-production-firmware)

---

## 1. Goal of Step 15A

Confirm, on the physical ELECROW door-side board, that the documented pinout in
`hardware/elecrow_pinout.md` matches the hardware revision in hand, and that
each door-side subsystem (display, touch, encoder, backlight, LED ring) is
electrically reachable on the GPIOs we plan to use.

This step is **only** about verifying the pinout source-of-truth and getting a
clean go/no-go signal for door-side hardware. It is **not** production firmware,
not the LVGL UI, not Home Assistant integration, not VL53L4CD work, not sensor
fusion.

This step implements the actions defined in
`docs/13_Firmware_Prep_Validation_Plan.md` §3.B
("ELECROW Board and Pinout Validation").

---

## 2. Hardware Being Validated

Door-side controller only:

- **Board:** ELECROW CrowPanel 1.28 in ESP32-S3 round rotary touch display
- **Display:** GC9A01A — 240 × 240 round, SPI, `ili9xxx` driver family in ESPHome
- **Touch:** CST816 (CST816D variant on this board) — I²C, expected address `0x15`
- **Rotary encoder:** quadrature encoder + push-button
- **Backlight:** PWM-controlled (LEDC)
- **RGB LED ring:** WS2812, 5 LEDs

Bedside / APDS-9960 / VL53L4CD / TSL2591 / SHT45 are **out of scope** for
Step 15A.

---

## 3. What Must Be Confirmed Before Any YAML / Firmware Work Begins

Before we touch production YAML or production firmware for the door side, the
following must each return a written PASS:

1. The exact ELECROW board revision in hand is identified (silkscreen, revision
   marking, photo on file).
2. The pinout in `hardware/elecrow_pinout.md` matches that board revision, or
   the deltas are written down.
3. The bring-up YAML `esphome/door_side_rotary.yaml` actually compiles for this
   board (no production behavior change — just compile + flash).
4. The display lights up and shows a test pattern using `GC9A01A` on the
   currently-documented SPI pins.
5. The CST816 touch controller answers on the I²C bus at the documented address
   and produces coordinates when touched.
6. The rotary encoder reports rotation in both directions on the documented
   A/B pins.
7. The encoder push-button registers a press on the documented switch pin.
8. The backlight pin accepts PWM and can be swept 0 → 100 %.
9. The WS2812 ring lights all 5 LEDs from the documented data pin.
10. No GPIO collisions or strap-pin surprises (e.g. boot-mode pins) appear in
    the logs during the above.

Only after **all** of the above are confirmed do we proceed to Step 15B
(door-side sensors / production door YAML / LVGL UI work).

---

## 4. Pinout / Source-of-Truth Checklist

Source of truth for this step: `hardware/elecrow_pinout.md` (already in repo).

Values currently documented there must be re-verified against the official
ELECROW board documentation / schematic / approved upstream repo for the exact
board revision in hand. Where the value already exists in the repo, treat it as
"candidate, needs confirmation" — not as ground truth — until physically
checked.

Where a value is **not** already in `hardware/elecrow_pinout.md`, do **not**
invent it. Mark it as **TBD — verify from board docs / schematic / existing
approved repo source**.

| Subsystem | Function | Repo candidate (from `hardware/elecrow_pinout.md`) | Physical check |
| --- | --- | --- | --- |
| Display (GC9A01A, SPI) | SCLK | GPIO10 | ☐ confirmed on board |
| Display | MOSI | GPIO11 | ☐ confirmed on board |
| Display | MISO | not used | ☐ confirmed on board |
| Display | DC | GPIO3 | ☐ confirmed on board |
| Display | CS | GPIO9 | ☐ confirmed on board |
| Display | RST | GPIO14 | ☐ confirmed on board |
| Display | Backlight (PWM) | GPIO46 | ☐ confirmed on board |
| Touch (CST816, I²C) | SDA | GPIO6 | ☐ confirmed on board |
| Touch | SCL | GPIO7 | ☐ confirmed on board |
| Touch | INT | GPIO5 | ☐ confirmed on board |
| Touch | RST | GPIO13 | ☐ confirmed on board |
| Touch | I²C address | `0x15` | ☐ confirmed by I²C scan |
| Rotary encoder | Encoder A | GPIO45 | ☐ confirmed on board |
| Rotary encoder | Encoder B | GPIO42 | ☐ confirmed on board |
| Rotary encoder | Encoder switch | GPIO41 | ☐ confirmed on board |
| WS2812 LED ring | Data | GPIO48 | ☐ confirmed on board |
| WS2812 LED ring | LED count | 5 | ☐ confirmed on board |
| Power indicator | Power LED | GPIO40 | ☐ confirmed on board |
| Test IO | aux | GPIO4, GPIO12 | ☐ confirmed on board |
| Onboard OLED (if present) | SDA | GPIO38 | ☐ TBD — confirm presence on this revision |
| Onboard OLED (if present) | SCL | GPIO39 | ☐ TBD — confirm presence on this revision |
| Board revision | silkscreen / sticker | TBD — verify from board docs / schematic / existing approved repo source | ☐ photographed |

Any row that does **not** match the physical board must be recorded as a delta
in the validation results document (see Section 6), not silently "fixed" in
this doc.

---

## 5. Safe Validation Order

Execute strictly in this order. Stop and record a FAIL the first time
something does not behave as expected — do **not** keep stacking subsystems on
top of an unverified one.

1. **Visual + documentation pass (no power yet)**
   - Photograph board front + back, capture silkscreen / revision marking.
   - Compare physical headers and labeled pads to `hardware/elecrow_pinout.md`.
   - Record any deltas.

2. **Power-on baseline (no flashing yet)**
   - Power the board via its standard USB connector with a known-good 5 V / 2 A
     supply.
   - Confirm power LED behavior (GPIO40 indicator) and absence of brownout /
     overheat.

3. **Bring-up YAML compile**
   - Compile `esphome/door_side_rotary.yaml` only (no edits).
   - Do not flash yet. Goal: confirm pin map / components compile clean on this
     board target.

4. **Flash + serial log inspection**
   - Flash the compiled bring-up image.
   - Watch serial logs for: boot, I²C scan output, no panic / reboot loop, no
     strap-pin warnings.

5. **Display check (GC9A01A)**
   - Confirm display initializes (no SPI error).
   - Confirm LVGL bring-up page renders ("VelaDial" / "Press dial" labels from
     the existing bring-up YAML).
   - Confirm `invert_colors: true` is still appropriate for this panel; record
     if not.

6. **Backlight PWM check**
   - Sweep backlight from 0 % → 100 % via the existing `back_light` light
     entity.
   - Confirm visible brightness change and no flicker / audible whine.

7. **Touch check (CST816)**
   - Confirm I²C scan reports `0x15`.
   - Confirm touches generate coordinates and that the existing transform
     (`mirror_y: true`, `swap_xy: true`) maps correctly to the visible panel
     orientation. Record any required transform change.

8. **Rotary encoder rotation check**
   - Rotate CW and CCW; confirm the encoder count moves in opposite directions
     on the two motions.
   - Record direction; do not "fix" direction in production YAML in this step.

9. **Rotary encoder press check**
   - Press the encoder shaft; confirm `rotary_button` registers a press in
     logs.

10. **WS2812 LED ring check**
    - Drive all 5 LEDs to a known color from a temporary `light` entity bound
      to GPIO48 (do **not** commit this temporary entity to production YAML).
    - Confirm all 5 LEDs respond, in order, no dropped pixels.

11. **Stability soak**
    - Leave the board powered for at least 30 minutes with display +
      backlight + touch active.
    - Confirm no reboots, no I²C bus errors, no thermal issues.

If any step fails, **stop**. Record the failure in the validation results doc
and open a follow-up task — do not skip ahead.

---

## 6. Do-Not-Proceed Gate (explicit)

> 🛑 **Do not proceed to production firmware, production YAML, the 3-page LVGL
> UI, sensor wiring, Home Assistant action wiring, or any door-side feature
> work until Step 15A has been completed end-to-end and the results have been
> reviewed and approved by Hardik.**

Specifically, until this step is signed off, do **not**:

- Modify `esphome/door_side_rotary.yaml` for production behavior (UI pages,
  presets, brightness logic, HA service calls beyond what already exists).
- Create any new production YAML under `esphome/`.
- Begin door-side sensor (TSL2591, SHT45) validation.
- Begin VL53L4CD / VL53L0X work (separately gated by Step 13 §3.A).
- Begin LVGL Power / Brightness / Presets page implementation.
- Begin sensor fusion work of any kind. Sensor fusion is explicitly out of v1.

Results of this validation must be written into a single document
(`hardware/validation_results.md`, as defined in
`docs/13_Firmware_Prep_Validation_Plan.md` §7) and reviewed before the gate is
lifted.

---

## 7. Expected Next Step After This PR Is Reviewed

After this PR is reviewed and merged:

1. Hardik confirms which board revision is in hand and any deltas vs.
   `hardware/elecrow_pinout.md`.
2. A separate validation PR (Step 15B) will:
   - Create `hardware/validation_results.md` with the ELECROW results section
     filled in.
   - If any pin deltas were found, update `hardware/elecrow_pinout.md` in the
     same PR (no silent fixes to production YAML in this repo before that).
3. Only after Step 15B is reviewed and merged do we move on to door-side
   sensor validation (TSL2591 + SHT45) per
   `docs/13_Firmware_Prep_Validation_Plan.md` §3.C.

Production firmware work for the door side remains gated until all of: this
step (15A) passes, the door sensor validation passes, and HA integration
validation passes — as defined in `docs/13_Firmware_Prep_Validation_Plan.md`
§6.

---

## Document Control

**Version:** 1.0  
**Scope:** Door-side ELECROW board / pinout validation only.  
**Owner approval required:** Yes, before lifting the gate in Section 6.
