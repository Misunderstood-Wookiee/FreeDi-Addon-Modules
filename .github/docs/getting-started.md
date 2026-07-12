# Getting Started

## Prerequisites

Before using any module in this repository, confirm all of the following:

1. Your printer is flashed with FreeDi Firmware.
2. Your printer config is based on FreeDi base machine configuration files.
3. Klipper can see these module files via your include paths.

## Installation

1. Copy this repository into your Klipper config storage location.
2. Include the module file(s) you want in your main printer configuration.
3. Restart Klipper after adding includes.
4. Test each macro manually before wiring it into slicer automation.

## Recommended Validation Flow

1. Run each macro once from the console with safe/default parameters.
2. Confirm there are no unknown command errors.
3. Confirm expected `RESPOND` and display messages appear.
4. Add macros to slicer start G-code only after manual verification.

## Slicer Integration Notes

- Pass string values for booleans as `True` or `False` where required.
- Keep parameter names exact (for example `SMOOTH_TIME`, not `SMOOTHTIME`).
- If slicer variables are unavailable, use fixed literals in G-code.

## Documentation Conventions

- Parameter names use uppercase to match macro usage.
- Times are listed in milliseconds for `G4 P...` contexts.
- Example snippets are intended to be copy/paste friendly.
