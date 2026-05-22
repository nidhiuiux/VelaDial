# Document 06 — Implementation Plan

## Phase 1 — Local lighting foundation

Goal: prove that Home Assistant can control all bulbs locally.

Tasks:

1. Install or configure a local Tuya LAN control integration.
2. Add the five Tuya RGB bulbs.
3. Confirm on/off, brightness, and color control from Home Assistant.
4. Create the bedroom light group.
5. Disconnect internet temporarily and confirm local control still works.

Done when:

- Home Assistant can control all bulbs as one group.
- Local-only operation is verified.

## Phase 2 — Sensor hardware validation

Goal: confirm all sensor hardware is physically correct before any firmware work.

Tasks:

1. Confirm final physical placement:
   - TSL2591 on door-side ESP32-S3 (ELECROW rotary display node).
   - SHT45 on door-side ESP32-S3.
   - APDS-9960 on bedside ESP32-C6.
   - VL53L4CD on bedside ESP32-C6.
2. Confirm STEMMA QT / Qwiic wiring per node.
3. Confirm two separate I2C buses (door-side and bedside are independent ESP32 devices).
4. Confirm no first-build I2C address conflict:
   - Door-side: TSL2591 at `0x29`, SHT45 at `0x44`.
   - Bedside: APDS-9960 at `0x39`, VL53L4CD at `0x29`.
5. Confirm SHT45 is only one sensor on door-side (no second SHT45).
6. Confirm no TCA9548A multiplexer is needed for first build.

Done when:

- All sensors are physically wired to their correct nodes.
- I2C scan confirms expected addresses on each node.
- No address conflicts exist within either bus.

## Phase 3 — Bedside gesture prototype

Goal: validate the ESP32-C6 and APDS-9960 path before building an enclosure.

Tasks:

1. Wire the APDS-9960 over I2C.
2. Flash the bedside ESPHome YAML after updating local secrets.
3. Confirm the sensor appears in Home Assistant.
4. Test left/right gestures in the intended bedside position.
5. Tune proximity threshold and cooldown.

Done when:

- Left gesture reliably turns lights off.
- Right gesture reliably turns lights on.
- False triggers are rare enough for bedroom use.

## Phase 4 — VL53L4CD bring-up

Goal: validate VL53L4CD distance sensor on the bedside node.

Tasks:

1. Wire VL53L4CD to bedside I2C bus.
2. Confirm I2C detection at `0x29`.
3. Verify ESPHome support path before firmware implementation.
4. If native ESPHome support is unavailable for VL53L4CD, **pause and ask the owner for approval before any fallback**. Evaluation options (in priority order):
   - Maintained community component or custom C++ component for VL53L4CD.
   - **VL53L0X** as the verified ESPHome-supported ToF fallback only. VL53L0X is **not** the selected default; re-tune the 5–10 cm hold threshold for VL53L0X's higher ~30 mm minimum range.
   - Do **not** silently substitute VL53L0X. Do **not** assume **VL53L1X** as a verified ESPHome fallback; VL53L1X support is unverified (no official `vl53l1x` docs page and no `vl53l1x` component in the ESPHome dev branch at the time of writing).
5. Test distance readings at 5 cm, 10 cm, 30 cm, and normal pass-by motion.
6. Tune hand-near threshold.
7. Tune hand-hold threshold around 5–10 cm for about 1.5 seconds.
8. Confirm quick pass-by does not trigger nightlight.

Done when:

- VL53L4CD reports stable distance readings.
- Hand-near detection is reliable and intentional.
- Quick pass-by motion is ignored.

Approval gate: No production YAML until sensor support path is verified. VL53L4CD remains the **planned** purchased bedside ToF sensor; the architecture is **not** being changed to VL53L0X.

## Phase 5 — Bedside v1 firmware implementation (no fusion in v1)

Goal: implement v1 production bedside behavior. v1 is **APDS-9960 standalone left/right gestures + VL53L4CD standalone hand-hold nightlight (only if VL53L4CD support is verified)**. Sensor fusion is **not** part of v1 — see Phase 5C.

### Validation gate (before any production bedside YAML)

- Phase 3 APDS-9960 standalone gesture readings verified in the real bedside mounting.
- Phase 4 VL53L4CD ESPHome support path verified (or a fallback explicitly approved by the owner per Phase 4 task 4).
- I2C scan on the bedside node confirms expected addresses (`0x39` APDS-9960 + `0x29` VL53L4CD).
- Actual mounting position tested for sunlight / ambient-IR false triggers and typical hand-approach angle.
- Cooldowns and quick-pass-by rejection validated during bring-up.
- **Do not implement sensor fusion in v1.**

### Phase 5A — APDS-9960 standalone gesture implementation (v1)

Tasks:

1. APDS-9960 owns directional gestures on its own. VL53L4CD does **not** arm or gate the gesture sensor in v1.
2. Left gesture turns the bedroom light group off.
3. Right gesture turns the bedroom light group on.
4. About 1 s cooldown after each successful gesture to prevent repeat-fire.
5. Ignore unclear, ambiguous, or noisy gesture reads. APDS-9960 misses are expected in real-world use.
6. Confirm sensor behavior does not cause accidental full-brightness light changes from noise.

Done when:

- Left/right gestures reliably control the bedroom light group in the real bedside mounting.
- Cooldown prevents repeated triggers.
- No accidental full-brightness changes from pass-by or noise.

### Phase 5B — VL53L4CD standalone hand-hold nightlight (v1, only if support verified)

**Gated on Phase 4 VL53L4CD support verification.** If support is blocked and a fallback to VL53L0X was approved by the owner per Phase 4 task 4, treat this phase as VL53L0X with re-tuned thresholds for its higher ~30 mm minimum range.

Tasks:

1. VL53L4CD handles deliberate hand-hold detection on its own. It does **not** arm APDS-9960.
2. Trigger: hand stable in the 5–10 cm range above the bedside sensor.
3. Hold duration: continuous in-range for about 1.5 s.
4. Hand-near debounce: require the hand to be in range for at least about 200 ms before the 1.5 s hold timer starts (prevents quick pass-by from starting the timer).
5. Cooldown: about 5 s after a successful trigger before another nightlight trigger is possible.
6. Action on trigger: Home Assistant turns the bedroom group on at warm low nightlight (about 10 % brightness, warm color).
7. Quick pass-by motion must not trigger.
8. Confirm sensor behavior does not cause accidental full-brightness light changes from noise.

Done when:

- Deliberate hand-hold at 5–10 cm for ~1.5 s reliably triggers the nightlight.
- Quick pass-by motion is ignored.
- Cooldown prevents repeated triggers.

### Phase 5C — Sensor fusion (v2 / future, not first-build requirement)

Sensor fusion is **not required for the first firmware build.** It is described here only as a future enhancement path.

- Future: VL53L4CD distance reading could be used to arm, wake, or refine APDS-9960 gesture detection (for example, only accepting APDS-9960 gesture events while VL53L4CD reports a hand within an arming range).
- Only add fusion after both Phase 5A (APDS standalone gestures) and Phase 5B (VL53L4CD standalone hold) are independently validated to work reliably in the actual bedside mounting.
- See `docs/03_App_Flow.md` and `docs/08_Sensor_Expansion_Research.md` for the broader fusion design discussion.

## Phase 6 — Door-side hardware bring-up

Goal: confirm display, touch, encoder, and backlight are working.

Tasks:

1. Flash the door-side ESPHome YAML to the ELECROW display.
2. Confirm boot, WiFi, API connection, and OTA.
3. Verify the GC9A01A display output.
4. Verify CST816 touch coordinates and transform.
5. Verify rotary encoder direction and button input.

Done when:

- The display renders a basic UI.
- Touch and encoder events are visible in logs.

## Phase 7 — TSL2591 bring-up

Goal: validate ambient light sensing on the door-side node.

Tasks:

1. Wire TSL2591 to door-side I2C bus.
2. Confirm I2C detection at `0x29`.
3. Confirm ESPHome TSL2591 sensor readings.
4. Record lux readings in dark bedroom, dim room, normal room, and bright room.
5. Tune lux-to-backlight mapping curve.
6. Confirm backlight changes smoothly and does not flicker.
7. Confirm TSL2591 controls display/backlight only, not room bulbs.

Done when:

- TSL2591 reports stable lux values.
- Backlight adapts smoothly to ambient conditions.
- Night brightness is capped.

## Phase 8 — SHT45 bring-up

Goal: validate temperature/humidity sensing on the door-side node.

Tasks:

1. Wire SHT45 to door-side I2C bus.
2. Confirm I2C detection at `0x44`.
3. Confirm ESPHome SHT4x temperature/humidity readings.
4. Expose values to Home Assistant.
5. Keep values secondary/diagnostic only.
6. Do not add required 4th Environment page.
7. Do not add HVAC/comfort automation in first build.

Done when:

- SHT45 reports reasonable temperature and humidity values.
- Values are visible in Home Assistant.
- No UI or automation depends on these values for first build.

## Phase 9 — Door-side control logic

Goal: connect physical input to lighting behavior, matching the locked behavior in `docs/03_App_Flow.md` and `docs/04_UI_UX_Design_Brief.md`.

Locked door-side behavior (must match):

- Main UI is exactly 3 pages: Power, Brightness, Presets. **No required 4th Environment page.**
- 4 first-build presets only: Warm White, Soft Amber, Neutral White, Low Nightlight. No dense color picker / color wheel / per-bulb editor; optional RGB accent is v2 / future only.
- Page navigation: horizontal swipe (left = next, right = previous) with a 3-dot page indicator near the bottom; active dot amber.
- Wake behavior is **wake-only first**: touch / knob rotation / knob press while asleep wake the display only; a second deliberate action while awake toggles or sends a command.
- Per-page knob press: Power = toggle the bedroom light group; Brightness = return to Power page; Presets = apply highlighted preset.
- Use "Unavailable" wording when Home Assistant / Pi / LAN state cannot be confirmed; subtle persistent badge only — no flashing, popups, audio, or full-screen aggressive errors.

Tasks:

1. Add power toggle behavior (tap and knob press on the Power page).
2. Add brightness arc and encoder binding (about 5 % steps, debounced).
3. Debounce encoder updates so commands feel smooth without flooding Home Assistant.
4. Add the 4 locked color preset actions (Warm White, Soft Amber, Neutral White, Low Nightlight) as 2×2 large touch targets with amber-bordered active preset.
5. Read current light state back from Home Assistant where possible and reconcile UI.

Done when:

- Door-side dial can turn the group on/off.
- Rotation adjusts group brightness predictably.
- Presets apply correctly.
- Page-swipe navigation and 3-dot indicator behave as locked.
- Wake-only-first behavior is honored.

Approval gate: No full LVGL integration until display bring-up and sensor readings are stable.

## Phase 10 — Adaptive display brightness implementation

Goal: make the display automatically adapt to room lighting conditions.

Tasks:

1. TSL2591 lux maps to display backlight target.
2. Bright mode for bright room.
3. Dim mode for moderate light.
4. Night mode for dark bedroom.
5. Smooth transitions.
6. Night brightness cap.
7. First touch/knob movement wakes screen, but night cap remains.
8. LED ring brightness may later follow ambient mode, but keep first build simple.

Done when:

- Display backlight adapts to ambient light automatically.
- Night brightness is comfortable and not disturbing.
- Transitions are smooth and flicker-free.

## Phase 11 — Night usability polish

Goal: make the device comfortable in a dark bedroom.

Tasks:

1. Tune idle dimming.
2. Keep fonts large and labels short.
3. Avoid full-screen bright states.
4. Add optional LED ring behavior only if it does not create distraction.
5. Test the device from real door and bedside positions.

Done when:

- The display is readable but not annoying at night.
- Waking the display does not accidentally change lights.

## Phase 12 — Validation and testing

Goal: confirm the full system works reliably under real conditions.

Tasks:

1. Test with internet disconnected.
2. Test Home Assistant local control.
3. Test dark bedroom behavior.
4. Test daylight readability.
5. Test accidental bump/wake behavior.
6. Test bedside gesture false positives.
7. Test nightlight hold reliability.
8. Test sensor failure fallback:
   - TSL2591 unavailable = fixed dim/sleep behavior.
   - SHT45 unavailable = hide diagnostics.
   - VL53L4CD unavailable = bedside falls back to APDS-9960 standalone gestures only (which is also the v1 standalone path); hold-nightlight feature disabled until VL53L4CD recovers.
   - APDS-9960 unavailable = bedside falls back to VL53L4CD standalone hold-nightlight only (if support is verified); directional gestures disabled until APDS-9960 recovers.

Done when:

- All primary flows work reliably.
- Sensor failures degrade gracefully.
- No accidental full-brightness light changes from sensor noise.

## Phase 13 — Enclosure and installation

Goal: install both controllers cleanly.

Tasks:

1. Design or print a small bedside enclosure with sensor visibility.
2. Design or print a wall faceplate for the rotary display.
3. Route safe 5V USB power.
4. Label cables and note final mounting details.
5. Run a full end-to-end test.

## Phase 14 — Documentation and hardware follow-up

Goal: document the final hardware configuration.

Tasks:

1. Create `hardware/sensor-wiring.md` (in a separate step, not this one).
2. Document separate door-side and bedside I2C buses.
3. Document sensor addresses.
4. Document STEMMA QT wiring order.
5. Document any GPIO needed only if later required.

Note: Do not create the hardware file in this implementation plan step.

## Approval gates

- No production YAML until sensor support path is verified.
- No full LVGL integration until display bring-up and sensor readings are stable.
- No PR until owner approves all doc updates.
- No merge by Manus.
- Main branch must not be touched.

## Final acceptance checklist

- APDS-9960 standalone left/right gestures work from the bed (v1).
- VL53L4CD standalone hand-hold nightlight works deliberately (v1, only if VL53L4CD support was verified).
- Sensor fusion (VL53L4CD arming / refining APDS-9960) is **not** required for first build; it is a v2 / future enhancement.
- Nightlight hold triggers only on deliberate hand-hold.
- Rotary brightness control works from the door.
- Display adapts to ambient light via TSL2591.
- Environmental data is available in Home Assistant.
- The system remains quiet during normal use.
- Internet disconnection does not break the main lighting workflow.
- Sensor failures degrade gracefully without disturbing sleep.
- Secrets are not committed to the repository.
