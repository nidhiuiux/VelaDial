# VL53L4CD ESPHome Support Verification

**Date:** 2026-05-22  
**Status:** **DECIDED — Option B (defer to v2)**, recorded 2026-05-25 by Hardik (owner).  
**Project:** VelaDial  

> **v1 decision (owner-recorded):** Hardik selected **Option B**. The
> VL53L4CD-based hand-hold nightlight feature is **deferred from v1** and
> moved to the v2 / future scope.
>
> v1 ships with **APDS-9960 standalone left/right gestures only** on the
> bedside node. The VL53L4CD sensor remains physically present on the
> bedside breadboard for future v2 work, but **no v1 firmware, ESPHome YAML,
> Home Assistant logic, or UI element references VL53L4CD**.
>
> Sensor fusion (VL53L4CD arming/refining APDS-9960) also remains a v2 /
> future enhancement — see `docs/03_App_Flow.md` § "Bedside sensor fusion".
>
> **VL53L0X fallback (Option C) is NOT approved.** No VL53L0X firmware,
> wiring, or doc changes shall be added unless Hardik explicitly revisits
> this decision.
>
> Re-opening this decision (e.g. for a v2 build with verified VL53L4CD
> support) requires:
> - A new dated `Status:` block in this section, AND
> - A new follow-up PR that updates: this doc, `docs/01_PRD.md` bedside
>   must-have list, `docs/05_Backend_Schema.md` bedside entities,
>   `docs/MASTER_EXECUTION_ROADMAP.md` Section H, and any firmware/setup
>   docs touched.



## 1. Purpose

The purpose of this document is to verify the ESPHome support path for the VL53L4CD Time-of-Flight (ToF) distance sensor. This verification acts as a necessary gate before proceeding to physical bedside ToF validation or attempting to write hold-nightlight firmware. The VL53L4CD remains the planned sensor for the bedside controller because it has been purchased. This document establishes whether the sensor can be used immediately or requires custom development or an approved fallback.

## 2. Sources Checked

The following sources were researched to determine current ESPHome support for the VL53L4CD sensor:
- Official ESPHome Sensor Documentation (`https://esphome.io/components/sensor/`)
- ESPHome GitHub Repository & `dev` branch (`https://github.com/esphome/esphome`)
- ESPHome Community forums, external component registries, and GitHub repositories
- Adafruit VL53L4CD library and product documentation (for context on custom implementation viability)

## 3. Official ESPHome Support Status

- **Status:** **Unsupported natively.**
- **Details:** There is no official `vl53l4cd` component listed in the current ESPHome component registry. Only `vl53l0x` and `vl53l1x` are referenced in various unofficial/official contexts, but the specific `vl53l4cd` model does not have a native integration.

## 4. ESPHome GitHub / Dev Branch Support Status

- **Status:** **Not found.**
- **Details:** A search of the ESPHome `dev` branch and open pull requests yields no active, ready-to-merge component for the `vl53l4cd`. Support is not imminent in the next immediate release based on the official repository.

## 5. Community / External Component Findings

- **Status:** **Limited / Requires custom wrap.**
- **Details:** While some developers have discussed the VL53L4CD on ESPHome community forums, there is no widely adopted, maintained "plug-and-play" external component available. Utilizing the sensor would likely require creating a custom C++ component within ESPHome that wraps the official STM32 or Adafruit VL53L4CD Arduino library.

## 6. Feasible Options

Based on the research above, the following are the feasible paths forward:

1. **Native Support:** Not possible at this time (component does not exist).
2. **Maintained External Component:** No reliable, actively maintained external component was found.
3. **Custom C++ Component:** Develop a custom ESPHome component that wraps the Adafruit VL53L4CD library. This requires C++ development and manual testing.
4. **Defer ToF Nightlight:** Ship the bedside controller in v1 with APDS-9960 standalone left/right gestures only. The nightlight feature would be delayed until the sensor is supported. (Sensor fusion is already excluded from v1).
5. **VL53L0X Fallback (with owner approval):** Switch to the VL53L0X sensor, which is natively supported. This is a fallback only and is not the selected default. It requires explicit approval and re-tuning of the hand-hold distance threshold due to its ~30mm minimum range limitation.

## 7. Recommendation

Given that the VL53L4CD has already been purchased and remains the planned sensor, I recommend **deferring the ToF nightlight feature temporarily** while investigating the effort required to build a **Custom C++ Component**. This prevents blocking the core v1 bedside capabilities (APDS-9960 gestures) while keeping the intended hardware intact. If writing a custom C++ component proves too complex for the current timeline, then explicit approval to fallback to the VL53L0X should be considered.

## 8. Decision Needed from Hardik

**RESOLVED 2026-05-25:** Hardik selected **Option B**.

- [ ] **Option A:** Proceed with building a custom C++ component for the VL53L4CD.
- [x] **Option B (SELECTED):** Defer the ToF nightlight feature for v1 (ship bedside with APDS-9960 only).
- [ ] **Option C:** Approve the VL53L0X fallback and adjust thresholds.

## 9. Impact on Next Steps

- **If Option A is chosen:** The next step is to research and write the custom C++ wrapper for the VL53L4CD, followed by Phase 5 ToF validation.
- **If Option B is chosen:** We skip Phase 5 ToF validation and proceed directly to APDS-9960 testing and bedside integration without the hand-hold nightlight feature.
- **If Option C is chosen:** The architecture will be updated to reflect the VL53L0X, and Phase 5 ToF validation will proceed using the VL53L0X parameters. 
- **Firmware restriction:** No ToF firmware will be written, and no sensor fusion will be implemented in v1 until this decision is finalized and the chosen path is validated.