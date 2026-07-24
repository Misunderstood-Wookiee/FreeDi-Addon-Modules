# FreeDi Add-on Modules Docs

This documentation covers all modules in this repository, their required context, and how to use them in slicer start G-code and manual macro calls.

## Important Compatibility Notice

- Caselight Effects requires FreeDi Firmware and FreeDi base machine configuration files.
- Dynamic Pressure Advance, Adaptive Line Purge, and Bed Soak are designed for FreeDi workflows but may also work on standard mainline Klipper/Kalico firmware depending on printer configuration.

For non-FreeDi stacks, treat these modules as use-at-your-own-risk and validate behavior before production prints.

## Docs Index

- [Getting Started](getting-started.md)
- [Dynamic Pressure Advance](modules/dynamic-pressure-advance.md)
- [Adaptive Line Purge](modules/adaptive-line-purge.md)
- [Bed Soak](modules/bed-soak.md)
- [Caselight Effects](modules/caselight-effects.md)

## Repository Layout

- `Adapative Pressure Advance/dynamic_pressure_advance_module.cfg`
- `Adapative Line Purge/adaptive_line_purge.cfg`
- `Bed Soak/bed_soak_module.cfg`
- `Caselight Effects/caselight_effects_core_module.cfg`
- `Caselight Effects/caselight_effects_sequences_module.cfg`

## Ongoing Documentation

These docs are the canonical GitHub documentation for this repository and will be continuously expanded as modules evolve.
