# Sensor Wiring Plan

## Purpose

This file documents the first-build sensor wiring plan for VelaDial. It covers the door-side ESP32-S3 rotary display node and the bedside ESP32-C6 gesture node.

## First-build architecture summary

### Door-side ESP32-S3 / ELECROW rotary display

- TSL2591 ambient light sensor
- SHT45 temperature/humidity sensor

### Bedside ESP32-C6

- APDS-9960 gesture sensor
- VL53L4CD time-of-flight distance sensor

## I2C architecture

Door-side and bedside nodes are separate ESP32 devices. Each node has its own independent I2C bus.

TSL2591 and VL53L4CD both use `0x29`, but there is no first-build conflict because they are on different ESP32 boards. Each node's I2C bus only has one device at `0x29`.

No TCA9548A I2C multiplexer is required for the first build. XSHUT/address reassignment is not required for the first build.

If both `0x29` sensors are ever moved to the same bus, the architecture must be revised.

## Door-side wiring

Node: ELECROW ESP32-S3 rotary display

| Sensor | Function | I2C Address | Connection | Notes |
| --- | --- | --- | --- | --- |
| TSL2591 | Ambient lux | `0x29` | STEMMA QT / Qwiic to door-side I2C bus | Used for display backlight only |
| SHT45 | Temperature/humidity | `0x44` | STEMMA QT / Qwiic daisy-chain on door-side I2C bus | Secondary diagnostics only |

Additional notes:

- Confirm exact available I2C expansion pins/connector on the ELECROW board before final wiring.
- Do not block display/touch I2C or SPI wiring.
- Keep TSL2591 placed where it can sense actual room/display ambient light, not hidden inside a dark enclosure.

## Bedside wiring

Node: ESP32-C6 bedside controller

| Sensor | Function | I2C Address | Connection | Notes |
| --- | --- | --- | --- | --- |
| APDS-9960 | Directional gestures | `0x39` | STEMMA QT / Qwiic to bedside I2C bus | Left/right gestures |
| VL53L4CD | Distance/hand hold | `0x29` | STEMMA QT / Qwiic daisy-chain on bedside I2C bus | Deliberate nightlight hold |

Additional notes:

- APDS-9960 and VL53L4CD should be mounted with clear sensor windows.
- VL53L4CD should face the hand approach zone.
- APDS-9960 should be positioned for natural left/right swipes.
- Avoid placing sensors where blankets/pillows will constantly trigger them.

## Suggested STEMMA QT daisy-chain order

### Door-side

```text
ESP32-S3 I2C → TSL2591 → SHT45
```

### Bedside

```text
ESP32-C6 I2C → VL53L4CD → APDS-9960
```

Order is not electrically critical for I2C, but the physical layout should reduce cable strain and keep sensors in useful positions.

## Validation checklist

### Door-side

- [ ] I2C scan shows TSL2591 at `0x29`.
- [ ] I2C scan shows SHT45 at `0x44`.
- [ ] TSL2591 lux changes when room light changes.
- [ ] SHT45 temperature/humidity values are reasonable.
- [ ] Display/touch/rotary still work after adding sensors.

### Bedside

- [ ] I2C scan shows APDS-9960 at `0x39`.
- [ ] I2C scan shows VL53L4CD at `0x29`.
- [ ] APDS-9960 detects left/right gestures.
- [ ] VL53L4CD distance changes with hand distance.
- [ ] Hand pass-by does not trigger nightlight.
- [ ] Stable hand hold at 5–10 cm can be detected.

## Enclosure notes

### Door-side

- TSL2591 needs exposure to room light.
- SHT45 needs airflow for accurate temp/humidity.
- Avoid sealing SHT45 in a warm enclosure near heat sources.
- Keep display readable and sensor openings subtle.

### Bedside

- APDS-9960 needs an IR-friendly window or direct exposure.
- VL53L4CD needs a clear optical path.
- Dark enclosure is preferred.
- Sensor placement must avoid false triggers from bedding or nearby objects.

## Safety notes

- Use 3.3V-compatible I2C logic.
- Confirm breakout board voltage requirements.
- Keep wiring strain-relieved.
- Do not connect/disconnect sensors while powered if avoidable.
- Do not route sensor wires where they can be pulled at night.
- Use short STEMMA QT cables when possible.

## Future options

- TCA9548A I2C multiplexer only if future architecture places conflicting addresses on the same bus.
- XSHUT/address reassignment only if VL53L4CD must share a bus with another `0x29` sensor.
- A separate environmental node is possible later but not part of first build.
- A second SHT45 near bed is not part of first build.
