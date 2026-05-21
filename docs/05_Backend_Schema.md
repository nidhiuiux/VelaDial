# Document 05 — Entity Model and Local Architecture

This project does not have a conventional web-app backend. Home Assistant is the local hub, and ESPHome devices communicate with it through the native API.

## Architecture

```text
Door-side ESPHome dial (ESP32-S3)
Bedside ESPHome sensor (ESP32-C6)
        ↓ WiFi LAN
Home Assistant
        ↓ local LAN integration
Tuya RGB bulbs
```

The ESPHome nodes are mostly stateless. They send user intent to Home Assistant and optionally display the current state.

## I2C bus architecture

Each ESPHome node has its own independent I2C bus. Sensors are connected via STEMMA QT / Qwiic daisy-chain within each node.

### Door-side node I2C bus

| Sensor | I2C Address | ESPHome Platform | Entity Type |
| :--- | :--- | :--- | :--- |
| TSL2591 | `0x29` | `tsl2591` (native) | Sensor (lux) |
| SHT45 | `0x44` | `sht4x` (native) | Sensor (temperature, humidity) |

### Bedside node I2C bus

| Sensor | I2C Address | ESPHome Platform | Entity Type |
| :--- | :--- | :--- | :--- |
| APDS-9960 | `0x39` | `apds9960` (native) | Binary sensor (gestures), sensor (proximity) |
| VL53L4CD | `0x29` | Needs verification | Sensor (distance in mm) |

TSL2591 and VL53L4CD both use address `0x29` but are on different ESP32 nodes, so there is no conflict in the first-build architecture.

## Home Assistant lighting model

### Light group

Recommended group:

- `light.bedroom_group`

Members:

- Five Tuya RGB bulb entities.
- Actual entity IDs may differ after LocalTuya setup.

Reason for grouping:

- One command controls all bulbs.
- ESPHome YAML stays simple.
- Future bulbs can be added in Home Assistant without changing firmware.

## ESPHome entities

### Door-side dial (ESP32-S3)

Expected entities:

- Rotary position sensor.
- Rotary button sensor.
- Touch wake event.
- Backlight control.
- Optional LED ring light entity.
- `sensor.room_ambient_light` — TSL2591 lux reading. Used internally for adaptive backlight PWM and exposed to Home Assistant for dashboard/automations.
- `sensor.room_temperature` — SHT45 temperature in °C. Exposed to Home Assistant for dashboard display and future automations.
- `sensor.room_humidity` — SHT45 relative humidity in %. Exposed to Home Assistant for dashboard display and future automations.

### Bedside gesture node (ESP32-C6)

Expected entities:

- Left gesture binary sensor.
- Right gesture binary sensor.
- Proximity sensor (APDS-9960).
- `sensor.bedside_distance` — VL53L4CD distance reading in mm. Used internally for hand-hold nightlight detection and exposed to Home Assistant for diagnostics.

Exact generated names depend on ESPHome naming and friendly-name settings.

## Derived entities and logic states

These are not raw sensor readings. They are computed states derived from sensor data, used to drive UI behavior, automations, or firmware logic.

### Door-side node derived states

| Derived State | Source | Logic | Used By |
| :--- | :--- | :--- | :--- |
| `display_brightness_target` | TSL2591 lux | Lux-to-PWM mapping curve. Low lux (< 5) → backlight at 10%. High lux (> 200) → backlight at 100%. Smooth interpolation between. | Backlight PWM output |
| `ambient_mode` | TSL2591 lux | `dark` if lux < 5, `dim` if 5–50, `bright` if > 50. | UI contrast/accent adaptation, LED ring intensity |
| `display_awake` | Touch event, rotary event, or timer | `true` after any input. Reverts to `false` after idle timeout (default: 60 s). | Backlight on/off, LVGL page visibility |
| `idle_timer` | Last input timestamp | Counts seconds since last user interaction. Resets on any touch, rotation, or press. | Display sleep logic |

### Bedside node derived states

| Derived State | Source | Logic | Used By |
| :--- | :--- | :--- | :--- |
| `hand_present` | VL53L4CD distance | `true` if distance < 300 mm for > 100 ms. `false` otherwise. | APDS-9960 wake trigger |
| `hand_holding` | VL53L4CD distance + timer | `true` if distance is 50–100 mm continuously for > 1.5 s. | Nightlight scene trigger |
| `gesture_ready` | `hand_present` state | `true` when `hand_present` is `true` and APDS-9960 is powered up and ready to detect. | Gesture detection gate |
| `nightlight_triggered` | `hand_holding` transition | One-shot `true` on rising edge of `hand_holding`. Resets after action is sent. Cooldown of 5 s prevents re-trigger. | HA service call: nightlight scene |
| `gesture_cooldown` | Last gesture timestamp | Blocks new gesture detection for 1 s after a successful gesture to prevent double-fires. | Gesture debounce |

### Home Assistant derived entities (optional, created via template sensors)

| Entity | Source | Logic | Purpose |
| :--- | :--- | :--- | :--- |
| `binary_sensor.bedroom_occupied` | VL53L4CD activity + gesture history | `true` if any bedside interaction in the last 30 minutes. | Future automations (auto-off after leaving) |
| `sensor.bedroom_comfort_index` | SHT45 temperature + humidity | Simple comfort formula (e.g., heat index or PMV approximation). | Dashboard display, future HVAC hints |

These Home Assistant template entities are optional and not required for the first build. They are documented here for future expansion.

### Example Home Assistant/template entity names

The following table lists the expected entity IDs as they will appear in Home Assistant after ESPHome integration. Actual names depend on ESPHome `name:` and `friendly_name:` settings, but these are the recommended defaults.

#### Door-side node entities (ESPHome device name: `veladial_doorside`)

| Entity ID | Type | Source | Update Rate | Notes |
| :--- | :--- | :--- | :--- | :--- |
| `sensor.veladial_doorside_ambient_light` | sensor | TSL2591 | 2 s | Lux value, 0–88000 range |
| `sensor.veladial_doorside_temperature` | sensor | SHT45 | 30 s | °C, ±0.1° accuracy |
| `sensor.veladial_doorside_humidity` | sensor | SHT45 | 30 s | %, ±1% accuracy |
| `sensor.veladial_doorside_backlight_level` | sensor | Internal | On change | Current backlight PWM % |
| `binary_sensor.veladial_doorside_display_awake` | binary_sensor | Internal | On change | Whether display is active |
| `sensor.veladial_doorside_wifi_signal` | sensor | WiFi | 60 s | dBm signal strength |

#### Bedside node entities (ESPHome device name: `veladial_bedside`)

| Entity ID | Type | Source | Update Rate | Notes |
| :--- | :--- | :--- | :--- | :--- |
| `binary_sensor.veladial_bedside_gesture_left` | binary_sensor | APDS-9960 | On event | Momentary pulse on left swipe |
| `binary_sensor.veladial_bedside_gesture_right` | binary_sensor | APDS-9960 | On event | Momentary pulse on right swipe |
| `sensor.veladial_bedside_proximity` | sensor | APDS-9960 | 200 ms | Raw proximity 0–255 |
| `sensor.veladial_bedside_distance` | sensor | VL53L4CD | 200 ms | Distance in mm, 0–1300 range |
| `binary_sensor.veladial_bedside_hand_present` | binary_sensor | VL53L4CD (derived) | On change | Hand detected within 300 mm |
| `binary_sensor.veladial_bedside_hand_holding` | binary_sensor | VL53L4CD (derived) | On change | Hand held at 50–100 mm for > 1.5 s |
| `sensor.veladial_bedside_wifi_signal` | sensor | WiFi | 60 s | dBm signal strength |

#### Home Assistant template entities (optional, configured in HA not ESPHome)

| Entity ID | Type | Template Logic | Required for First Build |
| :--- | :--- | :--- | :--- |
| `binary_sensor.bedroom_occupied` | binary_sensor | `true` if any bedside interaction in last 30 min | No |
| `sensor.bedroom_comfort_index` | sensor | Heat index from temperature + humidity | No |

## Actions

Initial Home Assistant actions:

- Turn the group off for a left gesture.
- Turn the group on for a right gesture.
- Turn the group on at low brightness and warm color for nightlight (triggered by VL53L4CD hand-hold at 5–10 cm for 1.5 s).
- Set brightness from the rotary control.

### Sensor-driven local behaviors (internal to ESPHome, not HA actions)

- TSL2591 lux reading → door-side backlight PWM adjustment (adaptive brightness).
- VL53L4CD distance → APDS-9960 wake trigger (sensor fusion on bedside node).

These behaviors run inside the ESPHome firmware and do not require Home Assistant round-trips.

## Sensitive data

Do not commit live WiFi credentials, API credentials, Home Assistant tokens, Tuya local keys, or generated ESPHome build output. Keep private values in the local ESPHome/Home Assistant configuration.

## Configuration values to keep easy to change

- Light group entity ID.
- Gesture threshold and cooldown.
- Screen dim timeout.
- Brightness step size.
- Color presets.
- Rotary encoder direction.
- Touch transform settings.
- TSL2591 lux-to-backlight mapping curve.
- VL53L4CD hand-hold distance threshold (default: 5–10 cm).
- VL53L4CD hand-hold duration threshold (default: 1.5 s).
- SHT45 update interval.

### Generic example entity aliases

| Generic Example Entity | Maps To / Derived From | Purpose |
| --- | --- | --- |
| `binary_sensor.bedside_hand_near` | `binary_sensor.veladial_bedside_hand_present` / VL53L4CD distance | Human-friendly generic alias for hand-near detection |
| `binary_sensor.bedside_nightlight_hold` | `binary_sensor.veladial_bedside_hand_holding` / VL53L4CD distance + timer | Human-friendly generic alias for deliberate nightlight hold |
| `number.display_auto_brightness_level` | `display_brightness_target` / TSL2591 lux mapping | Optional mapped display brightness target |
| `sensor.last_sensor_update` | Sensor timestamp diagnostics | Optional helper for stale sensor detection |
| `binary_sensor.sensor_health_ok` | Sensor availability checks | Optional helper for sensor health diagnostics |

These are example aliases/helper names only. Final production entity IDs depend on ESPHome names, Home Assistant helper setup, and firmware implementation.

## Sensor and entity behavior mapping

| Input/entity | Condition | Action |
| --- | --- | --- |
| TSL2591 lux | low ambient light | dim display/backlight |
| TSL2591 lux | bright room | increase display readability |
| VL53L4CD distance | hand steady 5–10 cm for about 1.5 s | trigger nightlight mode |
| APDS gesture left | valid gesture + cooldown clear | turn bedroom group off |
| APDS gesture right | valid gesture + cooldown clear | turn bedroom group on |
| SHT45 temp/humidity | available | expose diagnostics only |

## Security and privacy model

- No cloud backend is required.
- Sensor data stays inside Home Assistant/local network unless the owner later adds cloud integrations.
- No camera is used.
- No microphone is used.
- No audio recording is used.
- No voice dependency is used.
- Environmental data is diagnostic/comfort data only.

## Scope guard

- Temperature/humidity remains secondary.
- Environmental data must not clutter the main Power/Brightness/Presets UI.
- Environmental data is not part of the core lighting control loop.
- No HVAC/comfort automation is required in the first build.
- The main product remains a silent local-first lighting controller, not a general environmental monitor.

## Open implementation questions

- Confirm final ESPHome entity names after firmware compile.
- Verify VL53L4CD ESPHome support path.
- Decide exact lux-to-backlight mapping curve after real room testing.
- Decide exact distance threshold and hold time after bedside testing.
- Decide whether hand-near/nightlight-hold states live fully in ESPHome or are exposed to Home Assistant as helpers.
- Decide how often to publish SHT45 values to Home Assistant.
- Decide whether sensor health diagnostics are implemented in ESPHome or Home Assistant.
