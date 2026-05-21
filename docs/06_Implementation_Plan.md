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
4. If native support is unavailable, evaluate:
   - Custom component.
   - Community component.
   - Alternate ToF sensor such as VL53L1X.
5. Test distance readings at 5 cm, 10 cm, 30 cm, and normal pass-by motion.
6. Tune hand-near threshold.
7. Tune hand-hold threshold around 5–10 cm for about 1.5 seconds.
8. Confirm quick pass-by does not trigger nightlight.

Done when:

- VL53L4CD reports stable distance readings.
- Hand-near detection is reliable and intentional.
- Quick pass-by motion is ignored.

Approval gate: No production YAML until sensor support path is verified.

## Phase 5 — Bedside sensor-fusion implementation

Goal: combine VL53L4CD and APDS-9960 for reliable gesture and nightlight control.

Tasks:

1. VL53L4CD hand-near detection arms/wakes APDS-9960 gesture detection.
2. APDS-9960 remains responsible for left/right directional gestures.
3. Left gesture turns bedroom group off.
4. Right gesture turns bedroom group on.
5. Hand-hold (5–10 cm, ~1.5 s) triggers warm 10% nightlight.
6. Add cooldowns to prevent repeated triggers.
7. Confirm sensor behavior does not cause accidental full-brightness light changes.

Done when:

- Sensor fusion reliably distinguishes gesture from hold.
- Nightlight triggers only on deliberate hold.
- No accidental triggers from pass-by or bumps.

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

Goal: connect physical input to lighting behavior.

Tasks:

1. Add power toggle behavior.
2. Add brightness arc and encoder binding.
3. Debounce encoder updates so commands feel smooth without flooding Home Assistant.
4. Add color preset actions.
5. Read current light state back from Home Assistant where possible.

Done when:

- Door-side dial can turn the group on/off.
- Rotation adjusts group brightness predictably.
- Presets apply correctly.

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
   - VL53L4CD unavailable = APDS-only gesture fallback if possible.
   - APDS-9960 unavailable = ToF nightlight may still work if possible.

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

- Gesture control works from the bed.
- Sensor-fusion (VL53L4CD + APDS-9960) improves gesture reliability.
- Nightlight hold triggers only on deliberate hand-hold.
- Rotary brightness control works from the door.
- Display adapts to ambient light via TSL2591.
- Environmental data is available in Home Assistant.
- The system remains quiet during normal use.
- Internet disconnection does not break the main lighting workflow.
- Sensor failures degrade gracefully without disturbing sleep.
- Secrets are not committed to the repository.
