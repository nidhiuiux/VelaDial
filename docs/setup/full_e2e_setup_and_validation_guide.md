# VelaDial: Full E2E Setup and Validation Guide

## 1. Scope and Status

This document is the **End-to-End (E2E) validation guide only**. It defines the exact steps required to prove the VelaDial system works on physical hardware.

**Important Status Notes:**
- **This document does not prove testing has happened.**
- All results remain **NOT TESTED** until the user (Hardik) performs them on physical hardware.
- No secrets, logs, or binary evidence should be committed to the repository unless explicitly approved.

## 2. Full System Architecture

The VelaDial system operates entirely on the local network. The command paths are:

**Door-side Path:**
ELECROW ESP32-S3 → ESPHome → Home Assistant (Raspberry Pi) → LocalTuya (Local LAN) → `light.bedroom_group`

**Bedside Path:**
Adafruit ESP32-C6 + APDS-9960 → ESPHome → Home Assistant (Raspberry Pi) → LocalTuya (Local LAN) → `light.bedroom_group`

## 3. Prerequisites

Before beginning E2E validation, ensure the following are complete:

- PR #19 (Door-side draft), PR #20 (Bedside draft), and PR #21 (Raspberry Pi guide) are merged.
- Raspberry Pi and Home Assistant OS are installed and running.
- ESPHome add-on is installed in Home Assistant.
- LocalTuya is installed and configured with your Tuya bulbs.
- The `light.bedroom_group` entity is created and functional in Home Assistant.
- `secrets.yaml` is created in the `/config/esphome/` directory on Home Assistant (with real local values, **not committed to Git**).
- Physical hardware (ELECROW board, Adafruit board, APDS-9960) is available.
- High-quality USB data cables are available for initial flashing.

## 4. Hardware Checklist

Ensure you have the following physical components:

- ELECROW CrowPanel 1.28" ESP32-S3 rotary display (Door-side).
- Adafruit ESP32-C6 Feather (Bedside).
- APDS-9960 gesture sensor.
- Tuya-compatible lights/bulbs.
- Raspberry Pi 4 or 5 with official power supply and reliable storage.
- USB data cables (USB-C).
- Network router access (for IP reservations).

## 5. Safety and Secrets Rules

Strict adherence to these rules is required during validation:

- **No secrets in the repo:** Never commit `secrets.yaml`.
- **No Tuya local keys in the repo:** Keep them in Home Assistant only.
- **No Wi-Fi passwords in the repo.**
- **Redact logs:** Remove API keys, passwords, and external IP addresses from logs before sharing.
- **Raw evidence:** Photos and raw logs should be stored externally and linked, or explicitly approved before committing.

## 6. Flashing and Adoption Sequence

Follow this exact order to bring the devices online:

1. **Compile Door-side:** In the ESPHome dashboard, compile `esphome/door_side_rotary.yaml`.
2. **First Flash Door-side:** Connect the ELECROW board via USB and flash it using the ESPHome Web UI.
3. **Collect Boot Logs:** Immediately open the logs via USB to capture the initial boot sequence.
4. **Adopt in HA:** Once connected to Wi-Fi, Home Assistant should auto-discover the device. Click "Configure" to adopt it.
5. **Compile Bedside:** In the ESPHome dashboard, compile `esphome/bedside_gesture.yaml`.
6. **First Flash Bedside:** Connect the Adafruit ESP32-C6 via USB and flash it.
7. **Collect Boot Logs:** Capture the initial boot sequence via USB.
8. **Adopt in HA:** Adopt the bedside device in Home Assistant.
9. **Verify Online:** Ensure both devices show as "Online" in the ESPHome dashboard.
10. **Test Command Path:** Only proceed to E2E testing once both devices are online and adopted.

## 7. Door-side Validation

Use this table to validate the ELECROW door-side controller.

| Check | Expected Result | Actual Result | Status | Evidence / Notes |
| :--- | :--- | :--- | :--- | :--- |
| Power-on | Board powers up on USB 5V; no boot loop. | | NOT TESTED | |
| Board revision photo | Silkscreen/sticker photographed. | | NOT TESTED | |
| Front/back photos | Clear photos of the physical board. | | NOT TESTED | |
| ESPHome compile | Compiles without errors. | | NOT TESTED | |
| Flash | Flashes successfully via USB. | | NOT TESTED | |
| Serial boot log | Clean boot, no panics. | | NOT TESTED | |
| I2C scan | Detects touch controller at `0x15`. | | NOT TESTED | |
| GC9A01A display | Renders LVGL UI correctly. | | NOT TESTED | |
| Backlight PWM | Dims smoothly via HA entity. | | NOT TESTED | |
| CST816 touch address | Confirmed at `0x15`. | | NOT TESTED | |
| CST816 orientation | Touches map correctly to UI elements. | | NOT TESTED | |
| Rotary CW/CCW | Rotation registers correctly. | | NOT TESTED | |
| Rotary press | Press registers correctly. | | NOT TESTED | |
| WS2812 5 LED ring | LEDs light up (amber when lights on). | | NOT TESTED | |
| TSL2591 presence | Sensor detected and reports lux. | | NOT TESTED | |
| SHT45 presence | Sensor detected and reports temp/humidity. | | NOT TESTED | |
| Wake-only-first touch | Touch while asleep only wakes display. | | NOT TESTED | |
| Wake-only-first knob | Rotation while asleep only wakes display. | | NOT TESTED | |
| Wake-only-first press | Press while asleep only wakes display. | | NOT TESTED | |
| Power page toggle | Pressing knob toggles lights. | | NOT TESTED | |
| Brightness knob adjust | Rotation changes brightness in ~5% steps. | | NOT TESTED | |
| Brightness arc touch | **IMPLEMENTATION PENDING** (Visual only). | | NOT TESTED | |
| Presets page | 4 presets available and selectable. | | NOT TESTED | |
| Swipe navigation | Left/Right swipes change pages. | | NOT TESTED | |
| 3-dot indicator | Shows active page in amber. | | NOT TESTED | |
| 30-minute soak | Runs 30 mins without reboot or thermal issues. | | NOT TESTED | |

## 8. Bedside Validation

Use this table to validate the Adafruit ESP32-C6 bedside controller.

| Check | Expected Result | Actual Result | Status | Evidence / Notes |
| :--- | :--- | :--- | :--- | :--- |
| ESP32-C6 compile | Compiles without errors. | | NOT TESTED | |
| Flash | Flashes successfully via USB. | | NOT TESTED | |
| Boot log | Clean boot, no panics. | | NOT TESTED | |
| GPIO20 power enable | STEMMA QT port powered (HIGH). | | NOT TESTED | |
| I2C scan APDS-9960 | Detects sensor at `0x39`. | | NOT TESTED | |
| APDS left gesture | Registers LEFT gesture. | | NOT TESTED | |
| APDS right gesture | Registers RIGHT gesture. | | NOT TESTED | |
| 2-second cooldown | Prevents rapid repeated triggers. | | NOT TESTED | |
| Proximity diagnostic | Reports proximity values. | | NOT TESTED | |
| Wi-Fi/API status | Connects to HA reliably. | | NOT TESTED | |
| Uptime diagnostic | Reports uptime correctly. | | NOT TESTED | |
| No sensor fusion | Confirmed absent (v1 rule). | | NOT TESTED | |
| No VL53L4CD | Confirmed absent (v1 rule). | | NOT TESTED | |
| 30-minute soak | Runs 30 mins without reboot. | | NOT TESTED | |

## 9. Home Assistant / LocalTuya Command-Path Validation

Verify the core integration path.

| Check | Expected Result | Actual Result | Status | Evidence / Notes |
| :--- | :--- | :--- | :--- | :--- |
| `light.bedroom_group` | Entity exists and is available. | | NOT TESTED | |
| HA turn_on | Service call turns bulbs on. | | NOT TESTED | |
| HA turn_off | Service call turns bulbs off. | | NOT TESTED | |
| Brightness | Service call adjusts brightness. | | NOT TESTED | |
| Color temperature | Service call adjusts temp (if supported). | | NOT TESTED | |
| LocalTuya local control | Bulbs respond instantly. | | NOT TESTED | |
| Internet-off test | Control works with WAN disconnected (if safe). | | NOT TESTED | |
| Entity unavailable | Troubleshoot if Tuya IP changes. | | NOT TESTED | |
| Command latency | Note any noticeable delay. | | NOT TESTED | |

## 10. Full E2E Scenarios

Test the system as a user would interact with it.

| Scenario | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- |
| Door knob press | Toggles `light.bedroom_group`. | | NOT TESTED |
| Door brightness knob | Changes brightness of group. | | NOT TESTED |
| Door preset: Warm White | Applies Warm White to group. | | NOT TESTED |
| Door preset: Soft Amber | Applies Soft Amber to group. | | NOT TESTED |
| Door preset: Neutral White | Applies Neutral White to group. | | NOT TESTED |
| Door preset: Low Nightlight | Applies Low Nightlight to group. | | NOT TESTED |
| Bedside left gesture | Turns OFF `light.bedroom_group`. | | NOT TESTED |
| Bedside right gesture | Turns ON `light.bedroom_group`. | | NOT TESTED |
| Gesture cooldown | Rapid gestures are ignored (2s cooldown). | | NOT TESTED |
| Sleep/wake behavior | First input only wakes display, no command sent. | | NOT TESTED |
| HA restart recovery | Devices reconnect automatically after HA restart. | | NOT TESTED |
| Wi-Fi reconnect recovery | Devices reconnect automatically after Wi-Fi drop. | | NOT TESTED |
| RPi reboot recovery | System fully functional after RPi reboot. | | NOT TESTED |

## 11. Evidence Collection

To prove validation, collect the following (store externally, do not commit unless approved):

- **Logs to capture:** Initial USB boot logs for both devices, and a 5-minute log snippet during E2E scenario testing.
- **Where to capture:** Use the ESPHome dashboard "Logs" feature.
- **Photos needed:** Front and back of the ELECROW board, showing any revision stickers/silkscreen.
- **Screenshots:** Allowed only if redacted (no external IPs, no API keys).
- **Naming convention:** `veladial_door_bootlog.txt`, `veladial_bedside_bootlog.txt`, `elecrow_front.jpg`.
- **Storage:** Store in a shared Google Drive or similar external location.
- **Review:** Paste redacted log snippets back to ChatGPT/Claude for analysis if issues arise.

## 12. Failure Triage Matrix

If a test fails, consult this matrix.

| Symptom | Likely Cause | Where to Check | Next Action |
| :--- | :--- | :--- | :--- |
| HA not reachable | Network issue, RPi down | Router DHCP, RPi power/Ethernet | Reboot RPi, check IP |
| ESPHome compile fails | YAML syntax, missing secrets | ESPHome compile logs | Fix YAML/secrets |
| ESP32 not detected | Bad USB cable, missing drivers | OS device manager, dmesg | Swap cable, install drivers |
| OTA fails | Wrong OTA password, Wi-Fi drop | `secrets.yaml`, ESPHome logs | Verify password, move closer to AP |
| Door display blank | Backlight pin wrong, display init failed | `hardware/elecrow_pinout.md`, logs | Verify physical pins |
| Touch wrong orientation | Transform settings incorrect | `esphome/door_side_rotary.yaml` | Adjust `swap_xy` / `mirror_y` |
| Encoder direction reversed | A/B pins swapped | `esphome/door_side_rotary.yaml` | Swap GPIO45 and GPIO42 |
| APDS not found on I2C | Wiring, GPIO20 power off | Bedside logs, physical wiring | Verify GPIO20 is HIGH, check SDA/SCL |
| Gestures not triggering | Sensor blocked, I2C error | Bedside logs | Check physical sensor clearance |
| LocalTuya unavailable | IP changed, local key changed | HA Integrations, Router DHCP | Set static IP, re-extract key |
| Bedroom group missing | Not created in HA Helpers | HA Settings > Devices > Helpers | Create `light.bedroom_group` |
| Brightness not changing | LocalTuya DP mapping wrong | LocalTuya configuration | Reconfigure LocalTuya DPs |
| Color temp not supported | Bulb limitation | Tuya app | Use brightness/presets only |
| Wake-only-first violation | `resume_on_input` true, guard missing | `esphome/door_side_rotary.yaml` | Verify guard logic in YAML |
| Repeated gesture spam | Cooldown missing/too short | `esphome/bedside_gesture.yaml` | Increase `throttle` filter |
| Random reboots | Power supply weak, thermal issue | ESPHome logs (panic codes) | Use better USB power supply |

## 13. Acceptance Criteria

The system is **not accepted** until:

- Door compile PASS.
- Bedside compile PASS.
- Both devices flash and adopt successfully.
- Both devices remain online stably.
- HA/LocalTuya command path works reliably.
- Door-side hardware table is complete (all PASS or approved PARTIAL).
- Bedside hardware table is complete.
- All E2E scenarios complete successfully.
- 30-minute soak passed on both devices.
- All required evidence is collected and stored.
- No v1 rule is violated.
- No sensor fusion is added.
- No unsupported VL53L4CD claims are made.

## 14. Final Signoff Checklist

- [ ] Door-side PASS
- [ ] Bedside PASS
- [ ] HA path PASS
- [ ] LocalTuya PASS
- [ ] E2E PASS
- [ ] Known issues documented
- [ ] Owner (Hardik) approval
- [ ] Claude review complete

## 15. Explicit Non-Goals

- No firmware changes are included in this PR.
- No physical PASS claims are made in this document.
- No sensor fusion is implemented.
- No VL53L4CD implementation is included.
- No UI changes are made.
- No secrets, logs, or binaries are committed to the repository.
