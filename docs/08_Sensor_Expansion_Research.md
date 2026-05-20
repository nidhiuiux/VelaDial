# Document 08: Sensor Expansion Research & Integration Plan

**Author:** Manus AI
**Project:** VelaDial
**Date:** May 20, 2026

This document details the research, specifications, and integration strategy for the expanded sensor array in the VelaDial project. The addition of three new Adafruit sensors (TSL2591, VL53L4CD, SHT45) alongside the existing APDS-9960 significantly enhances the system's environmental awareness and interaction reliability.

## 1. Sensor Array Specifications

The VelaDial system now utilizes four distinct I2C sensors, daisy-chained via STEMMA QT connectors to the ESP32-C6 (bedside) and ESP32-S3 (door-side) microcontrollers.

| Sensor | Manufacturer Part | Primary Purpose | Key Specifications | I2C Address |
| :--- | :--- | :--- | :--- | :--- |
| **APDS-9960** | Broadcom | Directional gesture detection (Left/Right/Up/Down) | 10-20cm range, 2.4V-3.6V logic | `0x39` |
| **TSL2591** | Adafruit 1980 | High dynamic range ambient light sensing | 188 µLux to 88,000 Lux, 600M:1 range | `0x29` |
| **VL53L4CD** | Adafruit 5396 | Precise Time-of-Flight (ToF) distance measurement | 1mm to 1300mm range, 18° FoV | `0x29` (Default) |
| **SHT45** | Adafruit 6174 | Precision temperature and humidity monitoring | ±0.1°C, ±1% RH accuracy | `0x44` |

## 2. Critical Hardware Conflict: I2C Addressing

A significant hardware conflict exists in this array: **both the TSL2591 and the VL53L4CD share the default I2C address of `0x29`**. 

Because the TSL2591 has a fixed address that cannot be changed via software or jumpers, the conflict must be resolved using the VL53L4CD.

### Resolution Strategy
The VL53L4CD features an `XSHUT` (shutdown) pin. To resolve the conflict on a single I2C bus without a multiplexer:
1. Wire the `XSHUT` pin of the VL53L4CD to a dedicated GPIO pin on the ESP32.
2. On boot, pull the `XSHUT` pin LOW to disable the VL53L4CD.
3. Initialize the TSL2591 at `0x29`.
4. Pull the `XSHUT` pin HIGH to wake the VL53L4CD.
5. Immediately send an I2C command to reassign the VL53L4CD to a new address (e.g., `0x30`).
6. Initialize the VL53L4CD at its new address.

*Note: If STEMMA QT daisy-chaining prevents access to the `XSHUT` pin, an I2C multiplexer (like the TCA9548A) will be strictly required.*

## 3. Sensor Fusion & Interaction Strategy

The combination of these sensors allows for sophisticated "sensor fusion," where data from multiple sources is combined to create highly reliable interactions.

### Bedside Controller (ESP32-C6)
The bedside controller operates headlessly (no display) and relies entirely on physical gestures.

*   **Presence & Wake (VL53L4CD):** The ToF sensor continuously monitors the space above the nightstand. When a hand enters the 10cm-30cm range, it wakes the APDS-9960 from a low-power state.
*   **Directional Control (APDS-9960):** Once awake, the APDS-9960 looks for specific directional swipes. A left swipe turns the `light.bedroom_group` OFF; a right swipe turns it ON.
*   **Nightlight Mode (VL53L4CD):** If the ToF sensor detects a hand holding steady at a specific distance (e.g., 5cm-10cm) for more than 1.5 seconds, it triggers a "Nightlight" scene, setting the bulbs to 10% brightness and a warm color temperature. The ToF sensor is far more reliable for this static proximity detection than the APDS-9960.

### Door-Side Controller (ESP32-S3 + Rotary Display)
The door-side controller features the 1.28" LVGL display and utilizes environmental data to adapt its UI.

*   **Adaptive Brightness (TSL2591):** The TSL2591 continuously monitors ambient room light. During the day (high Lux), the display backlight is driven to 100% for visibility. At night (low Lux), the backlight dims significantly to prevent light pollution in the bedroom.
*   **Environmental Context (SHT45):** The SHT45 provides precision temperature and humidity data. While not controlling the lights directly, this data can be displayed on a secondary "Environment" page within the LVGL UI, adding premium value to the controller.

## 4. ESPHome Implementation Notes

*   **APDS-9960:** Supported natively via the `apds9960` platform. Requires careful tuning of `gesture_wait_time` and `proximity_threshold` to prevent false positives.
*   **TSL2591:** Supported natively via the `tsl2591` platform. Auto-gain configuration is recommended to handle the massive dynamic range between direct sunlight and pitch-black night.
*   **SHT45:** Supported natively via the `sht4x` platform.
*   **VL53L4CD:** This sensor requires a custom component in ESPHome, as it is not part of the standard library. Implementation will require importing a community-maintained custom sensor or writing a C++ wrapper for the Adafruit library.

## 5. Power Budget Considerations

The ESP32-C6 is designed for low power, but running four active sensors continuously can draw significant current. 
*   The VL53L4CD and APDS-9960 use active IR emitters.
*   To maintain a low power profile, the system should utilize the ESP32's sleep modes. The VL53L4CD can be polled at a lower frequency (e.g., 5Hz) until proximity is detected, at which point the APDS-9960 is powered up and polled at a high frequency (e.g., 50Hz) to catch the fast gesture.

## References
[1] Adafruit Industries. "Adafruit TSL2591 High Dynamic Range Digital Light Sensor."
[2] Adafruit Industries. "Adafruit VL53L4CD Time of Flight Distance Sensor."
[3] Adafruit Industries. "Adafruit SHT45 Precision Temperature & Humidity Sensor."
[4] ESPHome Documentation. "APDS-9960 Gesture, Proximity, Light and RGB Sensor."
[5] Home Assistant Community. "Adding Near/Far gesture control with APDS9960 ESPHome and HA."
