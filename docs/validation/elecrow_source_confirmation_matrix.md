# ELECROW Source Confirmation Matrix

**Status:** Source confirmation artifact (Pre-Step 15A)  
**Date:** 2026-05-23  
**Project:** VelaDial  
**Phase:** Validation preparation  

---

## 1. Purpose

This document cross-references the VelaDial repository's documented pinout (`hardware/elecrow_pinout.md`) and bring-up YAML (`esphome/door_side_rotary.yaml`) against the official Elecrow upstream repository. 

The goal is to confirm which GPIO assignments are backed by official upstream sources, which remain unconfirmed, and which require physical testing in Step 15A. This fulfills the "source-of-truth checklist" requirement before physical validation begins.

---

## 2. Upstream Sources Audited

The following official Elecrow sources were audited from the `Elecrow-RD/CrowPanel-ESP32-Display-Course-File` repository:

1. **`ESP32_Display_1_28.ino`** — Factory Arduino sketch
2. **`RotaryScreen_1_28_new.ino`** — Updated factory Arduino sketch
3. **`esphome1.28.yaml`** — Official Elecrow ESPHome reference configuration
4. **`128-lvgl-interface.yaml`** — Official Elecrow ESPHome LVGL lesson configuration
5. **`CST816D.h`** — Touch driver header
6. **`ESP32 Display-1.28-V1.0.pdf`** — Official Eagle schematic (V1.0)

---

## 3. GPIO Source Confirmation Matrix

The table below compares the VelaDial repository candidate values against the official upstream sources.

| Subsystem | Function | Repo Candidate | Upstream Source Value | Source File(s) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Display (SPI)** | SCLK | GPIO10 | GPIO10 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | MOSI | GPIO11 | GPIO11 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | MISO | not used | -1 (not used) | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | DC | GPIO3 | GPIO3 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | CS | GPIO9 | GPIO9 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | RST | GPIO14 | GPIO14 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | Backlight | GPIO46 | GPIO46 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| **Touch (I²C)** | SDA | GPIO6 | GPIO6 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | SCL | GPIO7 | GPIO7 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | INT | GPIO5 | GPIO5 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | RST | GPIO13 | GPIO13 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | Address | `0x15` | `0x15` | `esphome1.28.yaml`, `CST816D.h` | ✅ Source Confirmed |
| **Encoder** | Phase A | GPIO45 | GPIO45 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | Phase B | GPIO42 | GPIO42 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | Switch | GPIO41 | GPIO41 | `128-lvgl-interface.yaml`, `.ino` | ✅ Source Confirmed |
| **LED Ring** | Data | GPIO48 | GPIO48 | `.ino` | ✅ Source Confirmed |
| | Count | 5 | 5 | `.ino` | ✅ Source Confirmed |
| **Other IO** | Power LED | GPIO40 | GPIO40 | `esphome1.28.yaml`, `.ino` | ✅ Source Confirmed |
| | Aux IO | GPIO4, 12, 1, 2 | GPIO4, 12, 1, 2 | `esphome1.28.yaml` | ✅ Source Confirmed |
| | OLED SDA | GPIO38 | GPIO38 | `.ino` | ✅ Source Confirmed |
| | OLED SCL | GPIO39 | GPIO39 | `.ino` | ✅ Source Confirmed |

---

## 4. Configuration Parameters Confirmed

Beyond raw GPIO pins, the following ESPHome configuration parameters were confirmed against the official Elecrow upstream YAML:

1. **Display Model:** `GC9A01A` (Confirmed in `esphome1.28.yaml`)
2. **Color Inversion:** `invert_colors: true` (Confirmed in `esphome1.28.yaml` and `.ino`)
3. **Touch Transform:** `mirror_y: true`, `swap_xy: true` (Confirmed in `esphome1.28.yaml`)
4. **Encoder Pullups:** `pullup: true` on both A and B pins (Confirmed in `esphome1.28.yaml`)
5. **PSRAM Configuration:** `mode: octal`, `speed: 80MHz` (Confirmed in `esphome1.28.yaml`)

---

## 5. Items Pending Physical Validation (Step 15A)

While the source code confirms the intended design, the following items **must still be physically validated** on the actual board in hand during Step 15A:

1. **Board Revision:** The schematic indicates V1.0, but the physical silkscreen/sticker must be checked to ensure the board in hand matches the audited sources.
2. **Encoder Direction:** The physical wiring of Phase A vs Phase B determines which rotation direction registers as positive. This must be tested physically.
3. **Touch Orientation:** While the upstream YAML uses `mirror_y` and `swap_xy`, the physical mounting orientation of the display relative to the enclosure may require adjusting these transforms.
4. **LED Ring Order:** The physical layout of the 5 WS2812 LEDs (which is LED 0 vs LED 4) must be visually confirmed.
5. **OLED Presence:** The upstream code defines pins for an onboard OLED (`GPIO38`/`GPIO39`), but its physical presence on the specific board revision in hand must be verified.

---

## 6. Conclusion

The VelaDial repository's `hardware/elecrow_pinout.md` and `esphome/door_side_rotary.yaml` are **100% aligned** with the official Elecrow upstream sources. No discrepancies were found in the documented GPIO assignments.

The project is cleared to proceed to physical validation (Step 15A) using the existing bring-up YAML, with high confidence that the pin mappings are correct for the V1.0 board design.
