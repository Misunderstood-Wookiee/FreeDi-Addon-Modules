# Repository Overview

## What This Repository Contains

FreeDi-Addon-Modules contains reusable Klipper module files for FreeDi-based printers.

Included modules:

- Adaptive Pressure Advance
- Adaptive Line Purge
- Bed Soak
- Caselight Effects

## Required Platform Context

Compatibility guidance:

1. Caselight Effects requires FreeDi Firmware and FreeDi base machine configuration files.
2. Adaptive Pressure Advance, Adaptive Line Purge, and Bed Soak are designed for FreeDi workflows but may also work on standard mainline Klipper/Kalico firmware depending on your printer configuration.

For non-FreeDi stacks, treat these modules as use-at-your-own-risk and validate behavior before production prints.

## Module Source Files

- `Adapative Pressure Advance/adaptive_pressure_advance_module.cfg`
- `Adapative Line Purge/adaptive_line_purge.cfg`
- `Bed Soak/bed_soak_module.cfg`
- `Caselight Effects/caselight_effects_core_module.cfg`
- `Caselight Effects/caselight_effects_sequences_module.cfg`

## Documentation Maintenance

This GitHub docs set is maintained alongside module updates.
