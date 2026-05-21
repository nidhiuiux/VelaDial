# Document 02 — Technical Requirements

## System overview

VelaDial uses two ESPHome devices connected to Home Assistant over WiFi. Home Assistant owns the lighting entities and communicates with the Tuya bulbs locally. The ESPHome devices act as input surfaces and lightweight displays; they do not own long-term state.

## Hardware

### Door-side rotary display (ESP32-S3)

- ELECROW CrowPanel 1.28 in rotary display.
- ESP32-S3 class board.
- GC9A01A round IPS display, 240 x 240.
- Rotary encoder and capacitive touch input.
- Optional WS2812 LED ring.
- TSL2591 ambient light sensor, I2C address `0x29`. Purpose: ambient lux sensing for adaptive display backlight and day/night UI behavior.
- SHT45 temperature/humidity sensor, I2C address `0x44`. Purpose: room temperature and humidity diagnostics, secondary/future UI only.

### Bedside gesture controller (ESP32-C6)

- Adafruit ESP32-C6 Feather or equivalent ESP32-C6 board.
- APDS-9960 gesture/proximity sensor over I2C, address `0x39`. Purpose: directional gestures, used for left/right gesture control.
- VL53L4CD time-of-flight distance sensor over I2C, default address `0x29`. Purpose: reliable hand-near / hold detection for nightlight mode.
- Small bedside enclosure with the sensors exposed or behind a suitable window.

### Hub

- Home Assistant OS on Raspberry Pi 4/5 or another always-on local host.
- ESPHome add-on or ESPHome CLI for firmware build and OTA updates.
- HACS if LocalTuya is used.

## I2C architecture

The door-side node and bedside node are separate ESP32 devices with separate I2C buses.

### Door-side I2C bus (ESP32-S3)

| Sensor | I2C Address | ESPHome Platform |
| :--- | :--- | :--- |
| TSL2591 | `0x29` | `tsl2591` (native) |
| SHT45 | `0x44` | `sht4x` (native) |

### Bedside I2C bus (ESP32-C6)

| Sensor | I2C Address | ESPHome Platform |
| :--- | :--- | :--- |
| APDS-9960 | `0x39` | `apds9960` (native) |
| VL53L4CD | `0x29` | See note below |

TSL2591 and VL53L4CD both use `0x29`, but this is not a conflict in the first-build architecture because they are on different ESP32 nodes. XSHUT/address reassignment or a TCA9548A multiplexer is only a future fallback if both `0x29` sensors are ever placed on the same I2C bus.

All sensors connect via STEMMA QT / Qwiic connectors for solderless daisy-chaining within each node.

## Software stack

- ESPHome for firmware configuration and device integration.
- Home Assistant as the automation/control hub.
- LocalTuya or another local Tuya LAN integration for RGB bulb control.
- LVGL inside ESPHome for the round display UI.

## ESPHome support notes

- **TSL2591:** Natively supported via the `tsl2591` platform. Auto-gain recommended for bedroom use (handles both direct sunlight and pitch-dark conditions).
- **SHT45:** Natively supported via the `sht4x` platform.
- **APDS-9960:** Natively supported via the `apds9960` platform. Requires tuning of gesture sensitivity and proximity thresholds.
- **VL53L4CD:** ESPHome support needs verification before firmware implementation. If native support is unavailable, implementation may require a custom component, community component, or alternate ToF sensor.

## Home Assistant entities

Expected entities should stay configurable, but the first build assumes:

- `light.bedroom_group` — group containing the five RGB bulbs.
- Individual Tuya bulb entities may be named differently in a real installation.
- `sensor.room_ambient_light` — TSL2591 lux reading (door-side node).
- `sensor.room_temperature` — SHT45 temperature (door-side node).
- `sensor.room_humidity` — SHT45 humidity (door-side node).
- `sensor.bedside_distance` — VL53L4CD distance reading in mm (bedside node).

## Security and local configuration

Live network credentials, OTA credentials, API credentials, and Tuya local keys must stay outside the repository in the local ESPHome/Home Assistant configuration. The example firmware expects those values to be supplied by the local ESPHome secrets file.

## Functional requirements

- Left gesture turns the light group off.
- Right gesture turns the light group on.
- Proximity hold (VL53L4CD: hand steady at 5–10 cm for 1.5 s) triggers a warm 10% nightlight mode.
- Rotary encoder updates brightness in predictable increments.
- Touch UI supports power and preset selection.
- Display sleeps or dims after inactivity.
- Display backlight adapts automatically to ambient room light (TSL2591 lux → backlight PWM).

## Non-functional requirements

- Local LAN operation for normal use.
- No mechanical relay in the lighting control path.
- Fast response target: under 500 ms from user action to visible light change.
- Safe defaults after reboot.
- YAML should remain readable enough for manual debugging.

## Known implementation notes

- Pin mappings for the ELECROW display are tracked in `hardware/elecrow_pinout.md`.
- Display driver, touch transform, and encoder direction should be verified on real hardware.
- LocalTuya entity IDs and datapoints may vary by bulb model and firmware.
- VL53L4CD ESPHome integration status must be confirmed during firmware development. Fallback options include VL53L1X (native ESPHome support) or a custom C++ component.
