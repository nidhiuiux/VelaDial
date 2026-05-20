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

## Phase 2 — Bedside gesture prototype

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

## Phase 3 — Door-side hardware bring-up

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

## Phase 4 — Door-side control logic

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

## Phase 5 — Night usability polish

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

## Phase 6 — Enclosure and installation

Goal: install both controllers cleanly.

Tasks:

1. Design or print a small bedside enclosure with sensor visibility.
2. Design or print a wall faceplate for the rotary display.
3. Route safe 5V USB power.
4. Label cables and note final mounting details.
5. Run a full end-to-end test.

## Final acceptance checklist

- Gesture control works from the bed.
- Rotary brightness control works from the door.
- The system remains quiet during normal use.
- Internet disconnection does not break the main lighting workflow.
- Secrets are not committed to the repository.
