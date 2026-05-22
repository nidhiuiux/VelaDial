# Document 07 — Research Notes and Alternatives

This file records the options considered before settling on the current direction. Keep it practical: add findings, tradeoffs, links, and test notes as the build evolves.

## Tuya local control options

### LocalTuya — selected starting point

LocalTuya is a common Home Assistant approach for controlling Tuya devices over the local network using device local keys. It keeps the normal control path local and avoids cloud round trips.

Why it fits:

- Works with existing Tuya WiFi bulbs.
- Integrates directly into Home Assistant entities.
- Lets ESPHome control a simple Home Assistant light group.

Risks:

- Local keys can be annoying to retrieve.
- Tuya datapoints vary by product and firmware.
- Bulb firmware updates may change behavior.

### tinytuya

A Python library for direct local Tuya control.

Why it was not selected first:

- Adds a separate script or service layer.
- More custom maintenance than using Home Assistant entities.
- Less convenient for ESPHome-driven interaction.

### Tuya-CloudCutter / custom firmware

A route for replacing Tuya firmware on some devices.

Why it was not selected first:

- Many newer Tuya devices are patched or use chips that are harder to convert.
- Risk is higher for bulbs already working with local control.
- The project goal is a reliable room controller, not bulb firmware replacement.

### Tuya Local integration

A possible fallback if LocalTuya is not reliable for the exact bulb model.

## Bedside input alternatives

### APDS-9960 gesture sensor — selected starting point

Why it fits:

- Touchless.
- Silent.
- Small and inexpensive.
- Supports both gestures and proximity.

Risks:

- Ambient light, mounting angle, and hand distance can affect reliability.
- Needs careful debounce to avoid accidental triggers.

### Capacitive touch pad

Good fallback if gestures are unreliable.

Pros:

- Very quiet.
- Simple hardware.
- Can be hidden under or behind a surface.

Cons:

- Still requires touching a surface.
- Can be sensitive to humidity and mounting materials.

### Accelerometer tap sensor

Could detect a tap on a nightstand.

Reason not selected:

- Easier to trigger accidentally from movement or vibration.

### mmWave presence radar

Better for room presence than explicit bedside commands.

Reason not selected:

- Too likely to react to normal sleeping movement.

## Door-side display alternatives

### ELECROW 1.28 in rotary display — selected starting point

Why it fits:

- Built-in round display, touch, ESP32-S3, and rotary input.
- Good match for a compact wall controller.
- Enough UI capability without requiring a separate app.

### Cheap Yellow Display / generic ESP32 touchscreen

Reason not selected:

- Larger and less polished for a bedroom door control.
- Resistive touch variants feel less premium.
- No native rotary interaction.

### M5Stack Dial / similar premium dial hardware

Possible alternative if the ELECROW board creates driver or enclosure issues.

## UI framework options

### ESPHome + LVGL — selected starting point

Why it fits:

- Keeps firmware declarative and easy to update OTA.
- Integrates with Home Assistant naturally.
- Good enough for a small, focused interface.

### SquareLine Studio / custom LVGL C++

Reason not selected first:

- More powerful, but adds build and maintenance overhead.
- Less convenient for quick ESPHome iteration.

### openHASP

Worth revisiting for touchscreen-first dashboards.

Reason not selected first:

- Rotary behavior and the specific ELECROW hardware path are simpler to keep inside ESPHome for this project.

## Ambient light sensor alternatives

| Sensor | I2C Address | Dynamic Range | Best Use Case | ESPHome Support |
| --- | --- | --- | --- | --- |
| TSL2591 | `0x29` | 600M:1 (0.001–88,000 lux) | High dynamic range indoor/outdoor | Native (`tsl2591`) |
| BH1750 | `0x23`/`0x5C` | 1–65,535 lux | Simple indoor ambient light | Native (`bh1750`) |
| VEML7700 | `0x10` | 0–120,000 lux | General purpose ambient | Native (`veml7700`) |
| LTR-303 | `0x29` | 0.01–64,000 lux | Low-power ambient sensing | Native (`ltr303`) |
| TSL2561 | `0x29`/`0x39`/`0x49` | 0.1–40,000 lux | Legacy projects | Native (`tsl2561`) |

### TSL2591 — preferred for this build

Strengths: Extremely high dynamic range (600 million to 1). Excellent low-light sensitivity down to 0.001 lux. Separate visible and infrared channels allow accurate lux calculation even with mixed lighting. STEMMA QT / Qwiic available from Adafruit.

Weaknesses: Shares `0x29` with VL53L4CD (not a problem in this architecture since they are on different nodes). Slightly more expensive than BH1750.

Why it fits VelaDial: The bedroom ranges from near-total darkness (< 1 lux) to bright daylight (> 500 lux). TSL2591's low-light sensitivity is critical for smooth night-mode backlight adaptation.

### BH1750

Strengths: Very common, inexpensive, well-supported in ESPHome. Simple to use.

Weaknesses: Lower dynamic range. Less accurate in very dark conditions. May not distinguish between "very dark" and "pitch black" well enough for smooth night-mode transitions.

Why not preferred: Adequate for basic on/off brightness decisions, but TSL2591 provides better granularity in the critical low-light bedroom range.

### VEML7700

Strengths: Good range, low power, well-supported in ESPHome.

Weaknesses: Less low-light sensitivity than TSL2591. Slightly slower response time.

Why not preferred: A reasonable alternative if TSL2591 is unavailable, but TSL2591 is a better fit for the dark-bedroom use case.

### LTR-303

Strengths: Low power, good sensitivity, available on some Adafruit boards.

Weaknesses: Shares `0x29` address with TSL2591 and VL53L4CD. Not needed if TSL2591 is already used.

Why not preferred: Redundant with TSL2591 for this project.

### TSL2561

Strengths: Well-known legacy sensor.

Weaknesses: Older design, lower dynamic range than TSL2591. Shares address range with APDS-9960 at `0x39`.

Why not preferred: TSL2591 is the newer, better version.

### Conclusion

TSL2591 is the preferred first-build choice for door-side adaptive display brightness. It provides high dynamic range, excellent low-light sensitivity, and a good fit for day/night display adaptation. LTR-303 is not needed if TSL2591 is used. Ambient light sensing should control display/backlight behavior, not room bulbs directly.

Sources: Adafruit TSL2591 product page, ESPHome sensor documentation, Adafruit learning guides.

## Distance / proximity sensor alternatives

| Sensor | I2C Address | Range | Best Use Case | ESPHome Support |
| --- | --- | --- | --- | --- |
| VL53L4CD | `0x29` | 1–1300 mm | Close-range hand detection | Needs verification |
| VL53L0X | `0x29` | 30–2000 mm | General short-range ToF | Native (`vl53l0x`) |
| VL53L1X | `0x29` | 40–4000 mm | Medium-range ToF | Needs verification |
| VL6180X | `0x29` | 5–200 mm | Very close range + ambient light | Native (`vl6180x`) |
| APDS-9960 proximity | `0x39` | ~0–200 mm (unitless) | Basic near/far detection | Native (`apds9960`) |
| VCNL4040 | `0x60` | 0–200 mm | Proximity + ambient light combo | Community/custom |

### VL53L4CD — preferred for bedside hand-hold detection

Strengths: True time-of-flight distance measurement. Good close-range accuracy (1–1300 mm). Low power consumption. Small package. Provides real millimeter distance values rather than unitless proximity.

Weaknesses: ESPHome native support needs verification before firmware implementation. Shares `0x29` with TSL2591 (not a conflict in this architecture). Newer sensor with less community documentation than VL53L0X.

Why it fits VelaDial: Provides the precise distance data needed to distinguish "hand at 5–10 cm holding steady" from "hand passing by quickly." This enables reliable nightlight hold detection.

### VL53L0X — verified ESPHome fallback only (not selected default)

Strengths: Verified native ESPHome support — the `vl53l0x` platform exists in the current official ESPHome docs and in the ESPHome dev branch (see "Source verification note" below). Widely available. Good community documentation.

Weaknesses: Less accurate at very close range than VL53L4CD. Higher minimum range (30 mm vs 1 mm), which would require re-tuning the 5–10 cm hold-detection thresholds.

Why not preferred first: VL53L4CD was already purchased and has better close-range performance for the 5–10 cm hold detection use case.

Fallback positioning: VL53L0X is the verified ESPHome-supported ToF fallback **only**. It is **not** the selected default. Do not switch to VL53L0X unless VL53L4CD support validation blocks progress on the bedside hand-hold firmware and the project owner explicitly approves the swap. If used, hold thresholds must be re-tuned because of the higher 30 mm minimum range.

### VL53L1X

Strengths: Longer range (up to 4 m). Good close- to mid-range accuracy on paper.

Weaknesses: **ESPHome native support is unverified.** At the time of the latest review, no official `vl53l1x` ESPHome docs page and no `vl53l1x` component were found in the ESPHome dev branch (see "Source verification note" below). Optimized for longer range; may be less precise at the 5–10 cm hold distance. More expensive.

Why not preferred first: VL53L4CD is a better fit for the close-range bedside use case, and VL53L1X ESPHome support cannot be assumed.

Fallback note: VL53L1X should **not** be treated as a verified ESPHome fallback unless current official ESPHome documentation or a trusted maintained component source proves native or maintained support. Earlier notes that listed VL53L1X as natively supported should be considered out of date.

### VL6180X

Strengths: Very accurate at close range (5–200 mm). Also includes ambient light sensing.

Weaknesses: Maximum range only 200 mm, which limits hand-near detection distance. Older design.

Why not preferred: Range is too short for comfortable bedside interaction at 30 cm approach distance.

### APDS-9960 proximity

Strengths: Already on the bedside node. No additional hardware needed.

Weaknesses: Returns unitless proximity values (0–255), not real distance. Cannot reliably distinguish "hand at 5 cm" from "hand at 15 cm." Less suitable for deliberate hold detection.

Why not preferred for hold detection: APDS-9960 proximity alone is not preferred for deliberate hold detection because ToF gives real distance. APDS-9960 remains the preferred directional gesture sensor.

### VCNL4040

Strengths: Combined proximity and ambient light. Different I2C address (`0x60`).

Weaknesses: Limited range. Less community support. Not commonly available in STEMMA QT form.

Why not preferred: VL53L4CD provides better range and precision for this use case.

### Source verification note

The ESPHome-support column in the distance/proximity table above reflects the latest review. The following entries were checked directly against the current official ESPHome surface:

- **VL53L0X:** Verified — official ESPHome docs page exists (`esphome.io/components/sensor/vl53l0x.html`) and the `vl53l0x` component is present in the ESPHome dev branch.
- **VL53L4CD:** Not found at expected official locations — no `esphome.io/components/sensor/vl53l4cd.html` page and no `vl53l4cd` component in the ESPHome dev branch. A community / external component path is possible but was not verified in this review.
- **VL53L1X:** Not found at expected official locations — no `esphome.io/components/sensor/vl53l1x.html` page and no `vl53l1x` component in the ESPHome dev branch.

The other sensor rows in the table reflect earlier research and were not re-verified live in this review. Re-check official ESPHome docs and trusted maintained component sources before writing firmware that depends on any of them.

### Conclusion

VL53L4CD remains the **planned** purchased sensor for reliable bedside hand-near / hold detection because it was purchased and fits the intended 5–10 cm hand-hold interaction. APDS-9960 remains the preferred directional gesture sensor. APDS-9960 proximity alone is not preferred for deliberate hold detection because ToF gives real distance.

The architecture is **not** being changed to VL53L0X. VL53L4CD is not being abandoned. Firmware implementation of the VL53L4CD hand-hold path must wait until the ESPHome support path is verified. If verification blocks progress, evaluate **VL53L0X as the verified ESPHome-supported ToF fallback only**, after explicit owner approval, with hold thresholds re-tuned for VL53L0X's higher 30 mm minimum range. VL53L1X is **not** assumed to be a verified ESPHome fallback.

Sources: ST VL53L4CD datasheet, ESPHome VL53L0X documentation, Adafruit VL53L4CD product page. (Live ESPHome verification at the time of writing: VL53L0X docs and dev-branch component confirmed present; VL53L4CD and VL53L1X docs and dev-branch components not found at expected official locations.)

## Temperature / humidity sensor alternatives

### Temperature + humidity sensors

| Sensor | I2C Address | Temp Accuracy | Humidity | Best Use Case | ESPHome Support |
| --- | --- | --- | --- | --- | --- |
| SHT45 | `0x44` | ±0.1°C | ±1% RH | Premium indoor environment | Native (`sht4x`) |
| SHT40 | `0x44` | ±0.2°C | ±1.8% RH | Budget indoor environment | Native (`sht4x`) |
| BME280 | `0x76`/`0x77` | ±1.0°C | ±3% RH | Weather station / multi-sensor | Native (`bme280`) |
| AHT20 | `0x38` | ±0.3°C | ±2% RH | Budget humidity sensing | Native (`aht10`) |
| HDC3022 | `0x44` | ±0.1°C | ±0.5% RH | High-accuracy industrial | Needs verification |

### Temperature-only high accuracy sensors

| Sensor | I2C Address | Temp Accuracy | Best Use Case | ESPHome Support |
| --- | --- | --- | --- | --- |
| TMP119 | `0x48`–`0x4B` | ±0.1°C | Precision temperature logging | Needs verification |
| TMP117 | `0x48`–`0x4B` | ±0.1°C | Precision temperature reference | Native (`tmp117`) |
| MCP9808 | `0x18`–`0x1F` | ±0.25°C | General precision temperature | Native (`mcp9808`) |

### SHT45 — preferred for this build

Strengths: Excellent accuracy for both temperature (±0.1°C) and humidity (±1% RH). Fast response time. STEMMA QT / Qwiic available from Adafruit. Well-supported in ESPHome via `sht4x` platform. Small, reliable, low power.

Weaknesses: More expensive than budget alternatives. Provides more accuracy than strictly needed for secondary diagnostics.

Why it fits VelaDial: Already purchased. Provides high-quality temperature + humidity data in a simple I2C/STEMMA QT package. Good enough for future comfort display or automation if ever needed.

### SHT40

Strengths: Same family as SHT45, slightly lower accuracy, lower cost. Same ESPHome platform (`sht4x`).

Weaknesses: Slightly less accurate. Same I2C address as SHT45.

Why not preferred: SHT45 was already purchased and provides better accuracy at minimal cost difference.

### BME280

Strengths: Adds barometric pressure. Very common and well-supported. Different I2C address.

Weaknesses: Lower humidity accuracy (±3% RH). Known for humidity sensor drift over time. Larger package.

Why not preferred: Humidity drift is a known issue. Barometric pressure is not needed for VelaDial.

### AHT20

Strengths: Very inexpensive. Adequate accuracy for basic monitoring.

Weaknesses: Slower response. Less accurate than SHT45. Less reliable long-term.

Why not preferred: SHT45 is already purchased and provides better data quality.

### HDC3022

Strengths: Exceptional humidity accuracy (±0.5% RH). Industrial grade.

Weaknesses: ESPHome support needs verification. More expensive. Overkill for bedroom diagnostics.

Why not preferred: Overkill for this use case. ESPHome support uncertain.

### TMP119 / TMP117

Strengths: Excellent temperature accuracy (±0.1°C). TMP117 has native ESPHome support.

Weaknesses: Temperature only — no humidity. Overkill for secondary diagnostics.

Why not preferred: Do not provide humidity. SHT45 covers both temperature and humidity in one sensor.

### MCP9808

Strengths: Good temperature accuracy (±0.25°C). Well-supported in ESPHome.

Weaknesses: Temperature only — no humidity.

Why not preferred: Same reason as TMP117 — SHT45 covers both measurements.

### Conclusion

SHT45 is preferred for the first build because it provides high-quality temperature + humidity data in a simple I2C/STEMMA QT package. TMP119/TMP117 are excellent for pure temperature accuracy but do not provide humidity. Environmental data is secondary and should not drive the main lighting control path. No HVAC or comfort automation is required in the first build.

Sources: Sensirion SHT45 datasheet, ESPHome SHT4x documentation, Adafruit SHT45 product page.

## Sensor placement decisions

### First-build placement

| Node | ESP32 | Sensors | I2C Addresses |
| --- | --- | --- | --- |
| Door-side | ESP32-S3 (ELECROW rotary display) | TSL2591, SHT45 | `0x29`, `0x44` |
| Bedside | ESP32-C6 | APDS-9960, VL53L4CD | `0x39`, `0x29` |

### Placement rationale

TSL2591 belongs on the door-side node because it controls display/backlight behavior. The sensor should measure ambient light near the display it adapts.

SHT45 belongs on the door-side node because it supports future diagnostics/info display on the rotary screen. One sensor is sufficient for room-level temperature and humidity.

VL53L4CD belongs on the bedside node because it detects hand-near / hold behavior for nightlight triggering. It must be physically close to where the user's hand approaches.

APDS-9960 belongs on the bedside node because it detects left/right directional gestures for light on/off control.

## I2C architecture decision

Door-side and bedside nodes use separate I2C buses. They are independent ESP32 devices on the same WiFi network, each with their own I2C peripheral.

TSL2591 and VL53L4CD both use `0x29`, but this is not a first-build conflict because they are on different ESP32 nodes. Each node's I2C bus only has one device at `0x29`.

XSHUT pin reassignment or a TCA9548A I2C multiplexer is only a future fallback option. It would only be needed if both `0x29` sensors were ever placed on the same I2C bus, which is not part of the current architecture.

No I2C multiplexer is required for the first-build architecture.

## Research cautions

- Verify current ESPHome support before writing firmware. Sensor platform availability can change between ESPHome versions.
- Verify exact board addresses with I2C scan on each node before assuming defaults.
- Verify sensor placement physically before designing enclosures. Sensor orientation and mounting angle affect readings.
- Do not rely on sensor data for automatic full-brightness room-light changes. Sensor noise or failure must not disturb sleep.
- Keep sensor behavior silent and bedroom-safe. No audio feedback, no flashing LEDs, no aggressive responses to sensor events.

## Inspiration

- Smart thermostat rotary UI patterns.
- SmartKnob-style physical-to-visual control mapping.
- Minimal Home Assistant dashboards.
- Small nightstand devices that stay useful without becoming visually loud.
