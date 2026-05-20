# Agent Notes

This repository supports human-reviewed AI-assisted development.

## Source of truth

Read these files before proposing implementation changes:

1. `README.md`
2. `docs/01_PRD.md`
3. `docs/02_TRD.md`
4. `docs/03_App_Flow.md`
5. `docs/04_UI_UX_Design_Brief.md`
6. `docs/05_Backend_Schema.md`
7. `docs/06_Implementation_Plan.md`
8. `docs/07_Research_Alternatives.md`

## Rules for changes

- Keep the project local-first and silent.
- Do not add cloud-only control paths as required dependencies.
- Do not commit credentials or generated ESPHome build folders.
- Keep entity IDs, thresholds, presets, and timeouts easy to edit.
- Prefer small, reviewable changes over large rewrites.
- Mention any hardware assumption you make in the PR notes.

## Review checklist

Before calling a change complete, confirm:

- YAML uses `!secret` for credentials.
- The target Home Assistant light group remains configurable.
- Door-side UI still fits a 240 x 240 round display.
- Bedside gesture logic has debounce or cooldown behavior.
- Documentation matches firmware behavior.
