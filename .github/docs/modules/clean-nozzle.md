# Clean Nozzle

Source file: `X-Series 3/clean_nozzle.cfg`

## Quick Navigation

- [Macro](#macro)
- [Purpose](#purpose)
- [Parameters](#parameters)
- [Behavior Summary](#behavior-summary)
- [Typical Usage](#typical-usage)
- [Recommended PRINT_START Parameters](#recommended-print_start-parameters)
- [Notes](#notes)

## Macro

- `CLEAN_NOZZLE`

## Purpose

Performs a nozzle wipe over a configured brush path. The macro homes the printer if needed, heats the hotend to a safe wipe temperature, moves over the brush, and then optionally powers down the hotend or homes again depending on print state.

This script is intended for general Klipper machines that do not have any specific nozzle cleaning routine requirements, and it is also suitable for DIY nozzle installations on X-Series 3, Plus 3, Max 3, and X-Smart 3 printers. It is not intended for use with Q1-Pro, Plus 4, or machines that already use purge chute routines.

## Parameters

The macro exposes a set of configurable variables in the source file:

- `passes=<int>`: Number of wipe passes. Default is `4`.
- `wipe_temp=<int>`: Hotend target temperature used before wiping. Default is `180`.
- `min_wipe_temp=<int>`: Minimum actual nozzle temperature required before wiping. Default is `150`.
- `safe_travel_z=<float>`: Safe travel height before XY motion. Default is `20.0`.
- `approach_speed=<int>`: Speed for approach and reposition moves. Default is `15000`.
- `wipe_speed=<int>`: Speed for the actual wipe strokes. Default is `8000`.
- `start_pause_ms=<int>`: Pause after reaching the wipe start. Default is `50`.
- `wipe_pause_ms=<int>`: Pause between wipe strokes. Default is `10`.
- `powerdown_after_wipe=<int>`: Set to `1` to turn off the hotend after wiping when no print is active. Default is `1`.
- `home_after_wipe=<int>`: Set to `1` to home the printer after wiping when no print is active. Default is `1`.
- `LABEL=<string>`: Optional caller-provided context that extends the macro's `RESPOND` message. When supplied, the message becomes `CLEAN_NOZZLE: Wiping nozzle (<passes> passes) - <label>`.

## Behavior Summary

1. Validates the wipe configuration variables before moving.
2. Homes the printer if the axes are not already homed.
3. Heats the nozzle to the configured wipe temperature when needed.
4. Moves the toolhead to the configured wipe start position and performs the swish path.
5. Optionally powers down the hotend and re-homes after wiping when the printer is not actively printing or paused.
6. Restores the previous G-code state after the routine finishes.

## Typical Usage

```gcode
CLEAN_NOZZLE
```

With an additional label for startup or calibration context:

```gcode
CLEAN_NOZZLE LABEL="pre-calibration pass 1/2"
```

In this case, the macro appends the label to the status message so the console output clearly shows the caller context, for example:

```text
CLEAN_NOZZLE: Wiping nozzle (4 passes) - pre-calibration pass 1/2
```

## Recommended PRINT_START Parameters

This macro is best called explicitly from your startup flow when you want a wipe before calibration or probing. A typical pattern is:

```gcode
CLEAN_NOZZLE LABEL="pre-calibration nozzle wipe"
```

If your `PRINT_START` flow already includes a nozzle cleaning step, keep the call early so the wipe happens before the first probing or prime move.

## Notes

- The wipe path coordinates are configured in the module file and should be adjusted for your printer geometry.
- The macro is state-aware and avoids interfering with an active or paused print.
- The default configuration is intended for the X-Series 3 workflow and should be verified with your own printer limits before use.
