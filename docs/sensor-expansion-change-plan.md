# Sensor Expansion Change Plan

**Author:** Manus AI  
**Date:** May 20, 2026  
**Status:** CORRECTED — Awaiting owner approval to begin doc updates

> **Status note (added later):** This file is a historical planning artifact from the original sensor-expansion update (PR #1 era). It is preserved for traceability, but it is **not** the current source of truth for firmware behavior. Current decisions are owned by `docs/02_TRD.md`, `docs/03_App_Flow.md`, `docs/04_UI_UX_Design_Brief.md`, `docs/06_Implementation_Plan.md`, `docs/07_Research_Alternatives.md`, and `PROJECT_CONTEXT_FOR_AI.md`.
>
> In particular, any older wording in this file that implies VL53L4CD/APDS-9960 sensor fusion is a first-build requirement, or that VL53L4CD is definitely custom-component-only, should be treated as **superseded**. Current truth: APDS-9960 standalone left/right gestures are v1; VL53L4CD standalone hand-hold nightlight is v1 only if support is verified; VL53L4CD/APDS-9960 sensor fusion is v2 / future only; VL53L4CD ESPHome support path is **unverified** — not assumed native and not assumed custom-only.

This document describes the controlled set of changes required to integrate the three new Adafruit sensors (TSL2591, VL53L4CD, SHT45) into the VelaDial project documentation and firmware architecture. No changes will be made to the 7 source-of-truth docs until the owner explicitly approves each step.

---

## New Hardware Being Added

| Part | Adafruit # | Mouser # | I2C Address | Assigned Node | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| TSL2591 High Dynamic Range Light Sensor | 1980 | 485-1980 | `0x29` (fixed) | Door-side (ESP32-S3) | Ambient light sensing for adaptive display brightness and day/night UI adaptation |
| VL53L4CD Time-of-Flight Distance Sensor | 5396 | 485-5396 | `0x29` (default) | Bedside (ESP32-C6) | Reliable hand-near/hold detection for nightlight mode and display wake |
| SHT45 Precision Temp & Humidity Sensor | 6174 | 485-6174 | `0x44` | Door-side (ESP32-S3) | Bedroom comfort diagnostics, optional secondary info (not on main UI) |

---

## Corrected Architecture: Sensor-to-Node Assignment

### Door-Side Node (ESP32-S3 + ELECROW 1.28" Rotary Display)

| Sensor | I2C Address | Role |
| :--- | :--- | :--- |
| TSL2591 | `0x29` | Adaptive display backlight brightness based on ambient room light |
| SHT45 | `0x44` | Temperature/humidity data for optional diagnostics (not on main 3-page UI) |

### Bedside Node (ESP32-C6 + Headless Gesture Controller)

| Sensor | I2C Address | Role |
| :--- | :--- | :--- |
| APDS-9960 | `0x39` | Directional gesture detection (Left/Right/Up/Down) |
| VL53L4CD | `0x29` | Reliable hand-near/hold detection for nightlight mode |

### I2C Address Conflict: Resolved by Architecture

TSL2591 and VL53L4CD both use `0x29`, but this only creates a conflict if they are on the same I2C bus. The current recommended architecture avoids this conflict by placing TSL2591 on the door-side node and VL53L4CD on the bedside node.

**Future fallback (only if both `0x29` sensors are ever placed on the same bus):**
- Use the VL53L4CD XSHUT pin to reassign its address at boot, or
- Use a TCA9548A I2C multiplexer.

Neither is required for the first build.

---

## UI Scope Decision

The main door-side UI remains **3 pages only**:
1. Power
2. Brightness
3. Presets

Environmental data (temperature/humidity from SHT45) is **secondary** and will NOT be added as a 4th swipeable page. Future options include:
- Long-press access to a diagnostics view
- Idle mini-status overlay
- Home Assistant dashboard entity (primary way to view this data initially)

---

## Documents That Need Changes

### 01_PRD.md
- Add TSL2591, VL53L4CD, SHT45 to the hardware inventory section with their assigned nodes.
- Add "adaptive display brightness" as a product feature of the door-side controller.
- Add "reliable nightlight hold gesture via ToF" to bedside interaction requirements.
- Do NOT add "environment page" as a main feature. Mention it only as a future option.

### 02_TRD.md
- Add TSL2591, VL53L4CD, SHT45 to the hardware components table with correct node assignments.
- Document that the I2C address conflict is avoided by architecture (not by XSHUT/multiplexer).
- Add VL53L4CD custom component requirement (not natively supported in ESPHome).
- Update the I2C bus description to show two separate buses (door-side and bedside).
- Add power budget notes for each node's sensor load.

### 03_App_Flow.md
- Add bedside interaction: "VL53L4CD detects hand in range → wakes APDS-9960 → APDS reads gesture direction."
- Add nightlight hold interaction: "Hand steady at 5-10cm for 1.5s (VL53L4CD) → nightlight scene."
- Add door-side adaptive brightness flow: "TSL2591 lux reading → backlight PWM adjustment."
- Do NOT add a 4th page navigation flow.

### 04_UI_UX_Design_Brief.md
- Document the adaptive backlight behavior (bright in day, dim at night, driven by TSL2591).
- Note that TSL2591 data may influence amber accent intensity or UI contrast.
- Do NOT add a 4th "Environment" page to the main UI design.
- Mention temperature/humidity as a future secondary view only.

### 05_Backend_Schema.md
- Add new ESPHome sensor entities:
  - Door-side: `sensor.room_ambient_light` (TSL2591), `sensor.room_temperature` (SHT45), `sensor.room_humidity` (SHT45)
  - Bedside: `sensor.bedside_distance` (VL53L4CD)
- Add the VL53L4CD custom component dependency.
- Update the I2C bus architecture to show two separate nodes.

### 06_Implementation_Plan.md
- Add a sub-phase for sensor hardware assembly (STEMMA QT daisy-chaining per node).
- Add a testing step for verifying sensors communicate on each node's I2C bus.
- Add the VL53L4CD custom component integration step.
- Update the bedside firmware phase to include sensor fusion logic (VL53L4CD → APDS-9960 wake).
- Update the door-side firmware phase to include TSL2591 → backlight PWM logic.

### 07_Research_Alternatives.md
- Add a section comparing VL53L4CD to other ToF sensors (VL53L0X, VL53L1X, VL6180X).
- Add a section comparing TSL2591 to other light sensors (BH1750, VEML7700, TSL2561).
- Add a section comparing SHT45 to other temp/humidity sensors (SHT40, BME280, AHT20).

---

## Documents That Do NOT Need Changes

| File | Reason |
| :--- | :--- |
| `README.md` | Updated only after all doc changes are approved and merged. |
| `AGENTS.md` | Agent workflow is unchanged. |
| `PROJECT_CONTEXT_FOR_AI.md` | Regenerated after all changes are finalized. |
| `esphome/door_side_rotary.yaml` | Production firmware — not touched in this step. |
| `esphome/bedside_gesture.yaml` | Production firmware — not touched in this step. |
| `hardware/elecrow_pinout.md` | Covers the display only; unrelated to new sensors. |

---

## Files That Will Be Created (New)

| File | Purpose |
| :--- | :--- |
| `docs/08_Sensor_Expansion_Research.md` | Already created on this branch. Contains research findings, specs, and sensor fusion strategy. |
| `hardware/sensor-wiring.md` | New file documenting the I2C bus wiring per node, STEMMA QT daisy-chain order, and pin assignments. |

---

## Proposed Execution Order

| Step | Action | Scope |
| :--- | :--- | :--- |
| 1 | ~~Create change plan~~ | ~~Done~~ |
| 1B | ~~Correct change plan with owner answers~~ | ~~Done (this file)~~ |
| 2 | Owner reviews and approves corrected plan | — |
| 3 | Update `02_TRD.md` (hardware table, I2C architecture, power budget) | Controlled edit |
| 4 | Update `01_PRD.md` (hardware inventory, new features) | Controlled edit |
| 5 | Update `05_Backend_Schema.md` (new entities, bus architecture) | Controlled edit |
| 6 | Update `03_App_Flow.md` (new interactions, sensor fusion flow) | Controlled edit |
| 7 | Update `04_UI_UX_Design_Brief.md` (adaptive brightness, no 4th page) | Controlled edit |
| 8 | Update `06_Implementation_Plan.md` (new phases) | Controlled edit |
| 9 | Update `07_Research_Alternatives.md` (sensor comparisons) | Controlled edit |
| 10 | Create `hardware/sensor-wiring.md` | New file |
| 11 | Owner reviews all changes | — |
| 12 | Create PR for owner to merge (only when explicitly approved) | PR |

---

## Open Questions: RESOLVED

| # | Question | Owner Answer |
| :--- | :--- | :--- |
| 1 | TSL2591 placement? | **Door-side** (ESP32-S3). Near the display it controls. |
| 2 | SHT45 placement? | **Door-side** (ESP32-S3). One sensor only. No bedside SHT45 for first build. |
| 3 | XSHUT / TCA9548A needed? | **No.** Conflict avoided by architecture (sensors on different nodes). Future fallback only. |
| 4 | Environment page? | **No 4th page.** Main UI stays 3 pages. Temp/humidity is secondary/future. |
