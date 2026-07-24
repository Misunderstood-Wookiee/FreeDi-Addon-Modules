# Dynamic Pressure Advance

Source file: `Dynamic Pressure Advance/dynamic_pressure_advance_module.cfg`

## Quick Navigation

- [Macro](#macro)
- [Purpose](#purpose)
- [Parameters](#parameters)
- [Behavior Summary](#behavior-summary)
- [Start G-code Example](#start-g-code-example)
- [Recommended PRINT_START Parameters](#recommended-print_start-parameters)
- [Slicer Examples](#slicer-examples)
- [Debug and Testing (Before Printing)](#debug-and-testing-before-printing)
- [Notes](#notes)

## Macro

- `DYNAMIC_PRESSURE_ADVANCE`
- `ADAPTIVE_PRESSURE_ADVANCE` compatibility wrapper

## Purpose

Selects a pressure advance value using slicer input, material defaults, nozzle scaling, and a force override.

## Parameters

- `PRESSURE_ADVANCE=<float>`: Optional slicer-provided pressure advance.
- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PETG-CF`, `ABS`.
- `NOZZLE=<float>`: Nozzle diameter in mm. Default is `0.4`.
- `FORCE_PA=<bool-string>`: `True` or `False`. When `True`, ignores slicer pressure advance and uses module defaults.
- `SMOOTH_TIME=<float>`: Pressure advance smooth time. Default is `0.03`.

## Behavior Summary

1. Normalizes material family from `MATERIAL`.
2. Selects base default pressure advance by material.
3. Applies nozzle-based scaling factor.
4. Chooses slicer value unless `FORCE_PA=True`.
5. Validates pressure advance with a strict safe range of `> 0` and `<= 0.5`.
6. If slicer pressure advance is invalid (`<= 0` or `> 0.5`), logs a warning with `action_respond_info` and falls back to the detected material default (including nozzle scaling).
7. If the material-derived fallback is out of safe range, an emergency fallback of `0.042` is used.
8. If any computed pressure advance still ends up out of range, it is reset to the material-derived fallback with a warning.
9. Clamps smooth time to a safe range (`0.03` fallback if invalid, max `0.2`).
10. Applies values via `SET_PRESSURE_ADVANCE`.

## Start G-code Example

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE_PA=[force_pa] SMOOTH_TIME=[smooth_time]
```

If your slicer does not expose a smooth time variable:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE_PA=[force_pa] SMOOTH_TIME=0.03
```

## Recommended PRINT_START Parameters

Add these parameters to your `PRINT_START` macro if you want to drive this module from slicer start G-code:

- `MATERIAL`: Filament/material name from the slicer.
- `NOZZLE`: Nozzle diameter passed through from the slicer.
- `FORCE_PA`: Manual override flag, usually `False`.
- `SMOOTH_TIME`: Optional smoothing value. Use a fixed literal such as `0.03` if your slicer does not expose one.
- `PRESSURE_ADVANCE`: Optional. Pass this only if your slicer/profile already exposes a pressure advance variable. Otherwise omit it and let the module use its material-aware default.

This module only uses `FORCE_PA`. It does not read `FORCE_SOAK`.

For a full `PRINT_START` example showing parameter definitions for both modules together, see [Getting Started](../getting-started.md).

Typical `PRINT_START` usage inside Klipper:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
```

If your slicer can also provide pressure advance:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} PRESSURE_ADVANCE={pressure_advance} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
```

If you are using both Dynamic Pressure Advance and Bed Soak in the same `PRINT_START`, keep the force flags separate:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} PRESSURE_ADVANCE={pressure_advance} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
BED_SOAK MATERIAL={material} BED_TEMP={bed_target_temp} FORCE_SOAK={force_soak}
```

## Slicer Examples

These examples show what to add in slicer start G-code so the values are available to your Klipper `PRINT_START` macro.

For this module by itself, only `FORCE_PA` needs to be passed. `FORCE_SOAK` belongs to the Bed Soak module and should only be added if your `PRINT_START` also calls `BED_SOAK`.

### PrusaSlicer

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] FORCE_PA=False SMOOTH_TIME=0.03
```

If you maintain a slicer-side pressure advance variable in your profile, add it explicitly:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False SMOOTH_TIME=0.03
```

### OrcaSlicer

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] FORCE_PA=False SMOOTH_TIME=0.03
```

If you have a profile variable for pressure advance, pass it through explicitly:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False SMOOTH_TIME=0.03
```

### Cura

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL={material_type} BED_TEMP={material_bed_temperature_layer_0} EXTRUDER_TEMP={material_print_temperature_layer_0} NOZZLE={machine_nozzle_size} FORCE_PA=False SMOOTH_TIME=0.03
```

In Cura, pressure advance is usually better left to the Klipper module unless you already maintain a custom slicer variable for it.

## Debug and Testing (Before Printing)

Run these from the Klipper console (Mainsail/Fluidd) to validate pressure advance inputs and fallback behavior before launching a real print.

1. Confirm baseline material/nozzle default selection:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=PLA NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.03
```

Expected: module applies printer-side default for PLA with 0.4 nozzle scaling.

2. Confirm slicer-style pressure advance passthrough:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=PETG PRESSURE_ADVANCE=0.05 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.03
```

Expected: `SET_PRESSURE_ADVANCE` should use `0.05` when it is in the safe range.

3. Confirm invalid slicer value fallback path:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=PETG PRESSURE_ADVANCE=-1 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.03
```

Expected: warning message in console, then fallback to the material-derived value (not a hard failure).

4. Confirm smooth time validation:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=ABS PRESSURE_ADVANCE=0.06 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=1.0
```

Expected: smooth time is clamped/fallback-handled to a safe value per module logic.

5. Verify your full `PRINT_START` parameter mapping without printing a part:

```gcode
PRINT_START MATERIAL=PLA BED_TEMP=60 EXTRUDER_TEMP=210 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.03
```

Expected: no unknown-parameter errors, and `DYNAMIC_PRESSURE_ADVANCE` receives values exactly as your slicer will pass them.

Tip: Use `FORCE_PA=True` during diagnosis to isolate printer-side defaults from slicer-provided variables.

## Notes

- Keep slicer variable names aligned with your slicer profile.
- `ADAPTIVE_PRESSURE_ADVANCE` remains available as a compatibility wrapper for existing start macros.
- `FORCE_PA=True` is useful for enforcing printer-tuned defaults.
- Tune material defaults in the module file for your machine and filament.
- Invalid slicer PA values no longer hard-stop the macro; they are handled safely with warning messages and material-aware fallback.
