# VelaDial: Raspberry Pi & Home Assistant Setup Guide

## 1. Scope and Status

This document serves as the **setup documentation only** for the VelaDial system. It outlines the intended deployment path for Home Assistant, ESPHome, and LocalTuya.

**Important Status Notes:**
- **No physical validation has been completed.**
- The Home Assistant and LocalTuya command path is **NOT TESTED** until the user (Hardik) executes these steps on physical hardware.
- **No secrets should ever be committed** to the repository. All examples provided use placeholder values.

## 2. System Architecture

The VelaDial system relies on a local network architecture to ensure fast, cloud-independent control of bedroom lighting. The command path flows as follows:

**Door-side ESP32-S3 + Bedside ESP32-C6** → **ESPHome** → **Home Assistant (on Raspberry Pi)** → **LocalTuya (Local LAN)** → `light.bedroom_group`

This architecture ensures that physical interactions (knob turns, gestures) are processed locally by ESPHome, routed through Home Assistant, and delivered directly to the Tuya bulbs over the local network, minimizing latency.

## 3. Recommended Hardware

To ensure a stable and responsive system, the following hardware is recommended:

- **Raspberry Pi 5** (Preferred) or Raspberry Pi 4 (Acceptable, minimum 2GB RAM).
- **Official Raspberry Pi Power Supply** (Crucial for stability).
- **Reliable Storage:** High-endurance microSD card (Application Class 2 / A2) or an external USB SSD.
- **Network Connection:** Ethernet is highly preferred for the Raspberry Pi. Wi-Fi is acceptable but less ideal for a central hub.
- **Network Configuration:** Router DHCP reservation or static IP for the Raspberry Pi, ESP32 devices, and Tuya bulbs.
- **USB Cable:** A high-quality data cable for the initial flashing of the ESP32 devices.
- **VelaDial Hardware:**
  - ELECROW CrowPanel 1.28" ESP32-S3 Rotary Display (Door-side).
  - Adafruit ESP32-C6 Feather (Bedside).
  - APDS-9960 Gesture Sensor.
- **Lighting:** Tuya-compatible bulbs or lights already supported by the LocalTuya integration.

## 4. Installation Path

This guide assumes the use of **Home Assistant OS (HAOS)**, which is the official and recommended installation method for the Raspberry Pi. HAOS provides a managed environment that includes the Supervisor, making add-on installation (like ESPHome) straightforward.

If you choose to use a Supervised or Container installation later, the core concepts remain the same, but add-on management will differ.

## 5. First Boot and Access

1. **Flash the OS:** Use the Raspberry Pi Imager to write the Home Assistant OS image to your microSD card or SSD.
2. **Boot:** Insert the storage, connect the Ethernet cable, and plug in the power supply.
3. **Access:** Wait a few minutes, then open a web browser on a computer connected to the same network and navigate to `http://homeassistant.local:8123`.
   - *Fallback:* If the `.local` address does not resolve, check your router's DHCP client list to find the IP address assigned to the Raspberry Pi and navigate to `http://<IP_ADDRESS>:8123`.
4. **Onboarding:** Follow the on-screen prompts to create your admin user account and set your basic location and timezone.
5. **Update:** Before proceeding, navigate to **Settings > System > Updates** and ensure Home Assistant Core and OS are fully updated.

## 6. ESPHome Setup

ESPHome is used to compile and manage the firmware for the VelaDial hardware.

1. **Install Add-on:** In Home Assistant, go to **Settings > Add-ons**, click **Add-on Store**, search for "ESPHome", and install it. Start the add-on and enable "Show in sidebar".
2. **Open Web UI:** Click ESPHome in the sidebar to open the dashboard.
3. **Configuration Files:** The YAML files from this repository (`esphome/door_side_rotary.yaml` and `esphome/bedside_gesture.yaml`) must be placed in the `/config/esphome/` directory of your Home Assistant installation.
4. **Secrets:** Create a `secrets.yaml` file in the `/config/esphome/` directory. **Never commit real secrets to the repository.** Use the placeholder structure provided in Section 10.
5. **Expected Node Names:**
   - Door-side: `veladial-door-rotary`
   - Bedside: `bedside-gesture-controller`
6. **Compile Commands:** You can compile the firmware via the ESPHome Web UI or via command line if using a standalone ESPHome installation:
   - `esphome compile esphome/door_side_rotary.yaml`
   - `esphome compile esphome/bedside_gesture.yaml`
7. **Flashing:**
   - **First Flash:** Connect the ESP32 device to the computer running the ESPHome dashboard via USB. Use the Web UI to compile and install via the USB serial port.
   - **Subsequent Flashes:** Once the device is successfully connected to Wi-Fi and adopted by Home Assistant, future updates can be performed Over-The-Air (OTA).
8. **Logs:** Use the "Logs" button in the ESPHome dashboard to monitor device behavior and troubleshoot issues.

## 7. Required Home Assistant Entity

The VelaDial firmware is hardcoded to target a specific entity in Home Assistant:

`light.bedroom_group`

You must create this entity for the system to function.

1. **Verify Individual Lights:** Ensure your Tuya bulbs are integrated (via LocalTuya) and functioning individually in Home Assistant.
2. **Create Group:** Go to **Settings > Devices & Services > Helpers**.
3. **Add Helper:** Click **Create Helper**, select **Group**, and then select **Light group**.
4. **Configure:**
   - Name the group exactly: `bedroom group` (Home Assistant will automatically generate the entity ID `light.bedroom_group`).
   - Select the individual Tuya light entities to include in the group.
5. **Verify Actions:** Test the newly created `light.bedroom_group` from the Home Assistant dashboard. Ensure you can:
   - Turn it on and off.
   - Adjust brightness.
   - Adjust color temperature (if supported by the bulbs).

## 8. LocalTuya Setup

LocalTuya is a community integration that allows local control of Tuya devices, bypassing the cloud for faster response times.

1. **Install HACS:** If not already installed, install the Home Assistant Community Store (HACS).
2. **Install LocalTuya:** In HACS, search for and install the "LocalTuya" integration. Restart Home Assistant.
3. **Caution:** Note that LocalTuya is a community-maintained integration. Monitor it for updates and potential security or compatibility issues.
4. **Tuya Local Key:** You must obtain the `localkey` for each Tuya device. This typically involves creating a Tuya IoT developer account. Refer to community guides for the current extraction method. **Do not put Tuya keys in the repository.**
5. **Device Discovery:** LocalTuya often auto-discovers devices on the local network. If not, you will need the device IP address, ID, and local key to add it manually.
6. **Local Control Goal:** The primary goal is local LAN control. Cloud dependency should be minimized to initial key extraction only.
7. **Entity Mapping:** During LocalTuya setup, carefully map the Tuya Data Points (DPs) to the correct Home Assistant entity types (e.g., mapping the brightness DP to the brightness feature of a light entity).
8. **Troubleshooting:** If entities become unavailable, verify the device IP address hasn't changed (use DHCP reservations) and that the local key is still valid.

## 9. Network Reliability

A stable network is critical for the VelaDial system.

- **Static IPs:** Configure DHCP reservations in your router for the Raspberry Pi, both ESP32 devices, and all Tuya bulbs.
- **Network Segmentation:** Keep all devices on the same LAN/VLAN unless you are an advanced user comfortable configuring mDNS reflectors and firewall rules across subnets.
- **Wi-Fi Band:** Ensure the ESP32 devices and Tuya bulbs are connected to a stable 2.4GHz Wi-Fi network. They generally do not support 5GHz.
- **mDNS:** Home Assistant relies on mDNS for device discovery. Ensure your router does not block multicast traffic.

## 10. Secrets Handling

Create a `secrets.yaml` file in your `/config/esphome/` directory. **Do not commit this file to version control.**

A ready-to-copy template is tracked in the repository at
[`esphome/secrets.yaml.example`](../../esphome/secrets.yaml.example). Copy it
to `secrets.yaml` and fill in real values. The repo's `.gitignore` excludes
`secrets.yaml`, `esphome/secrets.yaml`, `**/secrets.yaml`, `*.local.yaml`,
`*.secret`, and `*.key` so a populated `secrets.yaml` won't be accidentally
committed — but never rely on that alone; double-check before pushing.

Sample `secrets.yaml` with placeholders:

```yaml
# /config/esphome/secrets.yaml
# DO NOT COMMIT REAL VALUES TO VERSION CONTROL

wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"

# 32-byte base64 encoded string for native API encryption
encryption_key: "YOUR_BASE64_ENCRYPTION_KEY_HERE="

# Password for Over-The-Air updates
ota_password: "YOUR_OTA_PASSWORD"

# Password for the fallback Wi-Fi access point
fallback_ap_password: "YOUR_FALLBACK_AP_PASSWORD"
```

*Note: Any LocalTuya keys must be entered directly into the Home Assistant UI during integration setup and must stay inside Home Assistant, not in the repository.*

## 11. Backup and Recovery

Regular backups are essential.

1. **Pre-HACS Backup:** Create a full backup in **Settings > System > Backups** before installing HACS or LocalTuya.
2. **Post-Setup Backup:** Create another full backup after successfully configuring all devices and the `light.bedroom_group`.
3. **Export:** Always download the backup file to a separate computer or cloud storage. Do not leave the only copy on the Raspberry Pi's SD card.
4. **Restore Process:** If the system fails, flash a fresh Home Assistant OS image, boot it, and select the option to restore from a backup during the initial onboarding screen.

## 12. Validation Checklist

Use this checklist to verify the deployment.

| Check | Expected Result | Actual Result | Status | Notes |
| :--- | :--- | :--- | :--- | :--- |
| HA boots | Web UI accessible at `homeassistant.local:8123` | | NOT TESTED | |
| ESPHome add-on installed | ESPHome dashboard accessible | | NOT TESTED | |
| Door YAML compiles | Successful compilation, no errors | | NOT TESTED | |
| Bedside YAML compiles | Successful compilation, no errors | | NOT TESTED | |
| Door device adopted | Device shows as "Online" in ESPHome | | NOT TESTED | |
| Bedside device adopted | Device shows as "Online" in ESPHome | | NOT TESTED | |
| `light.bedroom_group` exists | Entity available in HA states | | NOT TESTED | |
| HA service call turns lights on | Bulbs turn on via HA UI | | NOT TESTED | |
| HA service call turns lights off | Bulbs turn off via HA UI | | NOT TESTED | |
| Brightness command works | Brightness adjusts via HA UI | | NOT TESTED | |
| Color temp command works | Color temp adjusts via HA UI (if supported) | | NOT TESTED | |
| LocalTuya works locally | Bulbs respond instantly without internet | | NOT TESTED | |
| Door-side toggle reaches group | Pressing knob toggles `light.bedroom_group` | | NOT TESTED | |
| Bedside gestures reach group | Left/Right gestures toggle `light.bedroom_group` | | NOT TESTED | |
| Logs collected | ESPHome logs show successful API connection | | NOT TESTED | |

## 13. Troubleshooting

- **Cannot access HA:** Check Ethernet connection, verify IP address in router, ensure Raspberry Pi has adequate power.
- **ESPHome compile errors:** Verify YAML syntax, ensure `secrets.yaml` is present and correctly formatted.
- **ESP32 not detected over USB:** Try a different USB cable (ensure it supports data, not just charging), try a different USB port, install necessary serial drivers on your computer.
- **OTA fails:** Ensure the device is powered on, connected to Wi-Fi, and that the `ota_password` in `secrets.yaml` matches the one flashed to the device.
- **LocalTuya entities unavailable:** Verify device IP address, check if the local key has changed (happens if re-paired with the Tuya app).
- **Tuya bulbs not responding locally:** Ensure they are not blocked from the local network, verify LocalTuya configuration.
- **`light.bedroom_group` not found:** Double-check the entity ID in **Settings > Devices & Services > Entities**. It must be exact.
- **mDNS issues:** Check router settings for multicast or IGMP snooping options.
- **Wi-Fi drops:** Ensure strong 2.4GHz signal at the device locations.
- **API encryption mismatch:** Ensure the `encryption_key` in `secrets.yaml` matches what Home Assistant expects. Re-adopt the device if necessary.
- **Wrong secrets:** Double-check `secrets.yaml` for typos.
- **APDS gestures not triggering:** Check wiring (SDA/SCL), ensure I2C power pin is configured correctly, check ESPHome logs for I2C errors.
- **Door-side display/touch issues:** Refer to hardware validation documentation; verify pinouts and physical connections.

## 14. Acceptance Criteria

This setup guide is considered complete only when the user (Hardik) can successfully:

- Access the Home Assistant instance.
- Compile both ESPHome YAML configurations without errors.
- Flash and adopt both the door-side and bedside devices.
- Create the `light.bedroom_group` entity.
- Control the physical lights locally through the Home Assistant interface.
- Collect operational logs from the ESPHome devices.
- Proceed to run the full End-to-End (E2E) tests outlined in future phases.

## 15. Explicit Non-Goals

To maintain scope, the following are explicitly excluded from this guide and phase:

- No production hardware PASS claims are made.
- No sensor fusion logic is implemented or documented.
- No VL53L4CD implementation details are included.
- No UI changes to the door-side display are documented here.
- No firmware behavior changes are introduced.
- **No secrets are committed to the repository.**
