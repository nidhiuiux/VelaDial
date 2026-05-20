# ELECROW CrowPanel 1.28 in Rotary Display Pinout

Hardware reference for the ELECROW 1.28 in round touch rotary display used by VelaDial.

Verify this against the exact board revision before relying on it for production firmware.

## Display — GC9A01A over SPI

| Function | GPIO |
| --- | ---: |
| SCLK | 10 |
| MOSI | 11 |
| MISO | not used |
| DC | 3 |
| CS | 9 |
| RST | 14 |
| Backlight | 46 |

## Touch — CST816D over I2C

| Function | GPIO |
| --- | ---: |
| SDA | 6 |
| SCL | 7 |
| INT | 5 |
| RST | 13 |

## Rotary encoder

| Function | GPIO |
| --- | ---: |
| Encoder A | 45 |
| Encoder B | 42 |
| Encoder switch | 41 |

## RGB LED ring — WS2812

| Function | GPIO / Value |
| --- | ---: |
| Data | 48 |
| LED count | 5 |

## Other exposed pins

| Function | GPIO |
| --- | ---: |
| Power indicator | 40 |
| Test IO | 4, 12 |
| OLED SDA | 38 |
| OLED SCL | 39 |

## ESPHome notes

- Display model: `GC9A01A`.
- Display driver family: `ili9xxx`.
- Display resolution: 240 x 240.
- Touch platform: `cst816`.
- `invert_colors: true` may be required.
- Touch transform may need adjustment depending on physical rotation.

## Upstream reference

ELECROW publishes reference material under the CrowPanel 1.28 in HMI ESP32 rotary display project name. Use this file as a local working reference, not as a replacement for the official board documentation.
