# Document 13 — Firmware Prep Validation Plan

**Status:** Validation plan — hardware must be validated before production firmware  
**Date:** 2026-05-22  
**Project:** VelaDial

---

## 1. Purpose

This document defines the controlled validation sequence required before writing production ESPHome firmware for VelaDial.

**This is not production firmware.** This plan validates hardware, ESPHome component support, I2C addresses, and the Home Assistant control path before any production YAML is written.

**Existing YAML files are bring-up/starter configs only:**
- `esphome/door_side_rotary.yaml` — basic display/touch/encoder bring-up, not production UI
- `esphome/bedside_gesture.yaml` — basic APDS-9960 bring-up with known issues (60s update interval too slow for gestures)

This document supports `docs/06_Implementation_Plan.md` by providing a practical validation checklist. Production firmware requires validated hardware, confirmed component support, and verified I2C communication.

---

## 2. Validation Principles

- **Validate one subsystem at a time.** Do not combine display validation with sensor validation with HA integration in a single step.
- **Record pass/fail results.** Every validation step must produce a written result document.
- **Do not build full production UI until display/touch/encoder/backlight are validated.** The 3-page LVGL UI (Power, Brightness, Presets) comes after hardware validation.
- **Do not implement sensor fusion in v1.** APDS-9960 standalone gestures and VL53L4CD standalone hand-hold are v1. Fusion is v2/future only.
- **Do not write VL53L4CD hold-nightlight firmware until support path is verified.** VL53L4CD support verification must complete before bedside ToF firmware work.
- **Do not switch to VL53L0X unless VL53L4CD validation blocks progress and Hardik approves.** VL53L0X is verified fallback only, not selected default.

---

## 3. Validation Checklist

This checklist supports the phases in `docs/06_Implementation_Plan.md`. Complete each item and record results in `hardware/validation_results.md`.

### A. VL53L4CD Support Verification

**Goal:** Determine if VL53L4CD has a viable ESPHome support path.

**Actions:**

1. Check official ESPHome documentation at https://esphome.io/components/sensor/ for `vl53l4cd`
2. Check ESPHome GitHub repository for `vl53l4cd` component
3. Search trusted community/external components
4. **Decide one path:**
   - VL53L4CD native support
   - Maintained community component
   - Custom C++ component
   - Defer hold-nightlight (APDS-only bedside)
   - VL53L0X fallback only with owner approval

**Pass/Fail Criteria:**
- ✅ **PASS:** Native support confirmed, community component found, custom path feasible, or decision to defer
- ⚠️ **CONDITIONAL PASS:** VL53L0X fallback approved with documented threshold adjustments
- ❌ **FAIL:** No support path and no fallback approved

**Gate:** Bedside ToF validation cannot begin until this completes.

---

### B. ELECROW Board and Pinout Validation

**Goal:** Verify the physical ELECROW board matches documented pinout.

**Actions:**

1. Photograph board front/back, capture silkscreen and revision numbers
2. Compare to `hardware/elecrow_pinout.md`, note discrepancies
3. Validate each subsystem:
   - Display: GC9A01A initializes, shows test pattern
   - Touch: CST816 responds at `0x15`, generates coordinates
   - Encoder rotation: GPIO45/42 detect rotation, direction verified
   - Encoder press: GPIO41 detects button press
   - Backlight PWM: GPIO46 controls brightness 0-100%
   - WS2812 LED ring: GPIO48 lights up all 5 LEDs
   - I2C bus: Scan finds touch at `0x15`

**Pass/Fail Criteria:**
- ✅ **PASS:** All subsystems respond correctly, or issues documented with workarounds
- ✅ **PASS:** Board revision identified and pinout confirmed
- ⚠️ **PARTIAL:** Some GPIOs differ but workarounds documented
- ❌ **FAIL:** Critical GPIOs wrong and no workaround

**Gate:** Door-side sensors cannot be validated until this passes. Production door-side firmware cannot be written until this passes.

---

### C. Door-Side Sensor Validation

**Goal:** Verify TSL2591 and SHT45 function correctly on door-side I2C bus.

**Actions:**

1. Wire sensors via STEMMA QT (TSL2591 + SHT45)
2. Run I2C scan: expect `0x29` (TSL2591), `0x44` (SHT45), `0x15` (touch)
3. Test TSL2591: dark (<5 lux), dim (5-50), bright (>50), direct (>500)
4. Test SHT45: compare temp (±2°C) and humidity (±5%) to reference
5. Confirm sensors don't break display/touch/encoder

**Pass/Fail Criteria:**
- ✅ **PASS:** I2C scan finds `0x29` and `0x44`, readings appropriate, no interference
- ⚠️ **PARTIAL:** Sensors work with limitations (documented)
- ❌ **FAIL:** Sensors not detected or cause instability

**Gate:** Adaptive backlight firmware cannot be written until this passes.

---

### D. Bedside APDS-9960 Validation

**Goal:** Verify APDS-9960 gesture detection works reliably in bedside mounting position.

**Actions:**

1. Wire APDS-9960 via STEMMA QT to bedside ESP32-C6
2. Run I2C scan: expect `0x39`
3. Test proximity sensor (0-255 values)
4. Test gestures in actual mounting: left/right 10 attempts each, document success rate
5. Test environmental conditions: lights on/off, IR interference
6. Fix current issue: change `update_interval` from 60s to 50-100ms or interrupt-driven
7. Tune proximity threshold and gesture sensitivity
8. Add and validate ~1s cooldown after gesture

**Pass/Fail Criteria:**
- ✅ **PASS:** I2C finds `0x39`, left/right gestures ≥80% reliable, cooldown works
- ⚠️ **PARTIAL:** Gestures work with lower reliability (document conditions)
- ❌ **FAIL:** Gestures <50% reliable or false trigger constantly

**Gate:** Bedside APDS v1 firmware can be written after this passes. If this fails, consider button fallback or accept reduced reliability.

---

### E. Bedside ToF Validation

**Goal:** Verify VL53L4CD (or VL53L0X fallback) distance sensing and hand-hold detection.

**Prerequisite:** VL53L4CD support verification must complete first.

**Actions:**

1. Wire ToF sensor via STEMMA QT to bedside ESP32-C6
2. Run I2C scan: expect `0x29`
3. Test distance readings at 5cm, 10cm, 30cm, 100cm (±10mm acceptable)
4. If VL53L0X fallback: note 30mm minimum range, re-tune thresholds
5. Test hand-hold: 5-10cm for 1.5s triggers "holding", removal cancels, pass-by <200ms rejected, ~5s cooldown
6. Test for interference with APDS-9960

**Pass/Fail Criteria:**
- ✅ **PASS:** I2C finds `0x29`, readings accurate, hand-hold detection works, no interference
- ⚠️ **PARTIAL:** ToF works with limitations (documented)
- ❌ **FAIL:** Readings unstable or hand-hold detection unreliable

**Gate:** Bedside ToF nightlight firmware can be written after this passes. If this fails and VL53L0X fallback not approved, bedside ships APDS-only.

---

### F. Home Assistant Integration Validation

**Goal:** Verify ESPHome devices control HA lights via local network with acceptable latency.

**Actions:**

1. Confirm `light.bedroom_group` exists, controls all 5 bulbs
2. Confirm local control with internet disconnected
3. Measure response time: target <500ms
4. Test door-side: rotary → brightness, button → toggle
5. Test bedside APDS: left → off, right → on
6. Test nightlight (only if ToF validated): hand hold → nightlight scene

**Pass/Fail Criteria:**
- ✅ **PASS:** Group controls all bulbs, local control works, latency <500ms, commands reliable
- ⚠️ **PARTIAL:** Works but latency 500ms-1s (documented)
- ❌ **FAIL:** Commands don't reach bulbs or latency >1s

**Gate:** Production firmware development can begin after this passes.

---

## 4. Validation YAML Files

Validation YAML files will be created one phase at a time when needed. Do not create them now.

**Future validation YAMLs:**
- `esphome/door_hardware_validation.yaml` — ELECROW display, touch, encoder, backlight, WS2812
- `esphome/door_sensors_validation.yaml` — TSL2591 and SHT45
- `esphome/door_integration_test.yaml` — HA light control from door-side
- `esphome/bedside_apds_validation.yaml` — APDS-9960 gesture detection
- `esphome/bedside_tof_validation.yaml` — VL53L4CD or VL53L0X distance sensing
- `esphome/bedside_integration_test.yaml` — HA light control from bedside

Each YAML will be created only when its validation phase begins.

---

## 5. Risk Ranking

### 🔴 CRITICAL RISK

**VL53L4CD Support Risk**
- Impact: Bedside nightlight feature may be impossible without hardware swap
- Probability: High (official support unverified)
- Mitigation: Support verification checklist; VL53L0X fallback documented
- Consequence: Nightlight feature blocked if no support path and no fallback approved

### 🟠 HIGH RISK

**ELECROW Pinout Risk**
- Impact: Wrong GPIO assignments = non-functional hardware
- Probability: Medium (multiple board revisions exist)
- Mitigation: Physical validation checklist
- Consequence: PCB rework or board replacement needed

**APDS Gesture Reliability Risk**
- Impact: Unreliable gestures = poor UX, feature unusable
- Probability: Medium (gesture sensors finicky in real environments)
- Mitigation: Real-world testing; tune thresholds
- Consequence: May need button fallback or accept reduced reliability

### 🟡 MEDIUM RISK

**I2C/Address Risk**
- Impact: Sensors not detected or conflict
- Probability: Low (architecture separates `0x29` devices)
- Mitigation: I2C scans in validation checklists
- Consequence: Wiring error or defective sensor

**Network/HA Control Path Risk**
- Impact: Commands don't reach bulbs or latency too high
- Probability: Low (LocalTuya proven, LAN reliable)
- Mitigation: Integration validation checklist
- Consequence: Network/HA configuration debugging needed

**LVGL Performance Risk**
- Impact: UI lag, crashes, out-of-memory
- Probability: Low (8MB PSRAM sufficient for 240x240)
- Mitigation: Monitor heap during development
- Consequence: May need UI simplification

### 🟢 LOW RISK

**TSL2591/SHT45 Integration Risk**
- Impact: Ambient light or temp/humidity not working
- Probability: Very Low (native ESPHome support)
- Mitigation: Door sensor validation checklist
- Consequence: Display won't adapt; diagnostics missing (not critical)

**Power Supply Risk**
- Impact: Brownouts from insufficient power
- Probability: Very Low (sensors low power)
- Mitigation: Use quality 5V/2A+ supply
- Consequence: Intermittent crashes

---

## 6. Go/No-Go Gates

### Before door-side production UI work:
- ✅ ELECROW validation passed
- ✅ Door sensors validated (or documented as deferred)
- ✅ HA integration works with acceptable latency

**If any gate fails:** Debug and re-test before writing production UI. Do not build 3-page LVGL on unvalidated hardware.

---

### Before bedside APDS v1 firmware:
- ✅ APDS validation passed (gestures work reliably)
- ✅ HA integration works

**If APDS validation fails:** Consider physical button fallback or accept reduced reliability. Do not ship if gestures <50% reliable.

---

### Before VL53L4CD hold-nightlight firmware:
- ✅ VL53L4CD support verification passed or VL53L0X fallback approved
- ✅ ToF validation passed (hand-hold detection works reliably)
- ✅ HA integration works

**If support verification fails and no fallback approved:** Bedside ships APDS-only. Nightlight feature deferred.

**If ToF validation fails:** Bedside ships APDS-only. Nightlight feature deferred.

---

### Before production firmware:
- ✅ All relevant validation checklists passed
- ✅ Integration tests passed
- ✅ Owner review and approval of validation results
- ✅ Go/no-go decision made for any deferred features

**Door-side production firmware requires:** ELECROW validation, door sensors, HA integration  
**Bedside APDS production firmware requires:** APDS validation, HA integration  
**Bedside ToF production firmware requires:** VL53L4CD verification, ToF validation, HA integration

---

## 7. Validation Results Document

**Single consolidated result document:** `hardware/validation_results.md`

**This document will be created during validation phases. Do not create it now.**

**When created, it will contain subsections for:**
- VL53L4CD support verification results
- ELECROW board validation results
- Door-side sensor validation results
- Bedside sensor validation results
- HA integration test results

**Important:** This is the output of validation work, not an input. Create it incrementally as each validation phase completes.

---

## 8. Validation Workflow

**For each validation phase:**
1. Create branch: `firmware-prep/phase-N-description`
2. Create validation YAML (if needed for that phase)
3. Execute validation tests
4. Document results in `hardware/validation_results.md`
5. Commit result document
6. Create PR for owner review
7. Owner reviews and merges (or requests changes)
8. Proceed to next phase only after PR merged

**One phase at a time. One branch per phase. One PR per phase.**

**No production firmware until all relevant validation phases pass.**

---

## Document Control

**Version:** 1.0  
**Status:** Validation plan — approved for execution  
**Next step:** Execute VL53L4CD support verification  
**Owner approval required:** Before each phase transition
