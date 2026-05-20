# VelaDial

A quiet, local-first bedroom lighting controller built with Home Assistant, ESPHome, a round rotary display, and a bedside gesture sensor.

VelaDial controls a group of Tuya RGB bulbs without relying on voice commands, cloud latency, relay clicks, or a phone screen at night.

## Hardware concept

- Door-side ELECROW 1.28 in rotary touch display for power, brightness, and color presets.
- Bedside ESP32-C6 with APDS-9960 gesture sensor for silent hand-wave control.
- Home Assistant as the local hub.
- LocalTuya or an equivalent local LAN integration for the existing Tuya bulbs.

## Repository layout

```text
.
├── docs/
├── esphome/
└── hardware/
```

## Start here

1. Read the files in `docs/` first.
2. Confirm Home Assistant and local bulb control are working.
3. Review the ESPHome YAML files before flashing hardware.
4. Keep all credentials in ESPHome/Home Assistant `secrets.yaml`.

## Design principles

- Silent by default.
- Local-first control path.
- Low light pollution for bedroom use.
- Small, reviewable firmware and documentation changes.
- Useful for AI-assisted development, but still human-reviewed.

## ESPHome secrets

Create these in your local ESPHome `secrets.yaml`:

```yaml
wifi_ssid: "your_wifi_name"
wifi_password: "your_wifi_password"
fallback_ap_password: "temporary_setup_password"
ota_key: "your_ota_password"
encryption_key: "your_esphome_api_encryption_key"
```

Do not commit WiFi passwords, Tuya local keys, Home Assistant tokens, or generated ESPHome build output.
