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
- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PETG-CF`, `PA12-CF`, `PPS-CF`, `TPU-Aero`.
- `NOZZLE=<float>`: Nozzle diameter in mm. Default is `0.4`.
- `FORCE_PA=<bool-string>`: `True` or `False`. When `True`, ignores slicer pressure advance and uses module defaults.
- `SMOOTH_TIME=<float>`: Pressure advance smooth time. Default is `0.04`.

## Behavior Summary

1. Normalizes material family from `MATERIAL`.
2. Supports Bed Soak-aligned families and aliases, including `PCTG`, `PET`, `PP`, `PEBA`, `TPU_AIR`, `PPA`, `PPS`, `PA6`, `PA11`, `PA12`, and `PAHT`.
3. Selects base default pressure advance by material.
4. Applies nozzle-based scaling factor.
5. Chooses slicer value unless `FORCE_PA=True`.
6. Validates pressure advance with a strict safe range of `> 0.005` and `<= 0.5`.
7. If slicer pressure advance is valid but high (`> 0.25`), it is accepted and a warning is emitted for visibility.
8. If slicer pressure advance is invalid (`<= 0.005` or `> 0.5`), logs a warning with `action_respond_info` and falls back to the detected material default (including nozzle scaling).
9. If the material-derived fallback is out of safe range, an emergency fallback of `0.042` is used.
10. If any computed pressure advance still ends up out of range, it is reset to the material-derived fallback with a warning.
11. Clamps smooth time to a safe range (`0.04` fallback if invalid, min `0.02`, max `0.2`).
12. Applies values via `SET_PRESSURE_ADVANCE`.

## Material Default Buckets

Current default pressure advance map (before nozzle scaling):

- `PLA`: `0.042`
- `PETG`, `PCTG`: `0.086`
- `PET`, `PC/POLYCARBONATE`: `0.082`
- `ABS`, `ASA`, `PVA`, `HIPS`: `0.035`
- `TPU`, `TPU_AIR`: `0.150`
- `PEBA`: `0.130`
- `NYLON`, `PA6`, `PA11`, `PA12`, `PAHT`, `PPA`: `0.100`
- `PPS`: `0.090`
- `PP`: `0.060`
- `GENERIC`: `0.035`

Note: these values are machine-dependent safe defaults and should be tuned for your printer/extruder.

## Start G-code Example

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE_PA=[force_pa] SMOOTH_TIME=[smooth_time]
```

If your slicer does not expose a smooth time variable:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE_PA=[force_pa] SMOOTH_TIME=0.04
```

## Recommended PRINT_START Parameters

Add these parameters to your `PRINT_START` macro if you want to drive this module from slicer start G-code:

- `MATERIAL`: Filament/material name from the slicer.
- `NOZZLE`: Nozzle diameter passed through from the slicer.
- `FORCE_PA`: Manual override flag, usually `False`.
- `SMOOTH_TIME`: Optional smoothing value. Use a fixed literal such as `0.04` if your slicer does not expose one.
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
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] FORCE_PA=False SMOOTH_TIME=0.04
```

If you maintain a slicer-side pressure advance variable in your profile, add it explicitly:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False SMOOTH_TIME=0.04
```

### OrcaSlicer

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] FORCE_PA=False SMOOTH_TIME=0.04
```

If you have a profile variable for pressure advance, pass it through explicitly:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False SMOOTH_TIME=0.04
```

### Cura

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL={material_type} BED_TEMP={material_bed_temperature_layer_0} EXTRUDER_TEMP={material_print_temperature_layer_0} NOZZLE={machine_nozzle_size} FORCE_PA=False SMOOTH_TIME=0.04
```

In Cura, pressure advance is usually better left to the Klipper module unless you already maintain a custom slicer variable for it.

## Debug and Testing (Before Printing)

Run these from the Klipper console (Mainsail/Fluidd) to validate pressure advance inputs and fallback behavior before launching a real print.

### Compatibility Test Matrix

Use this matrix to validate material compatibility with Bed Soak families, alias mapping, wrapper compatibility, and safety paths.

| Goal | Command | Expected |
|---|---|---|
| Baseline default path | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PLA NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Uses PLA default bucket and nozzle scaling |
| Slicer passthrough in range | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PETG PRESSURE_ADVANCE=0.05 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.04` | Uses slicer value |
| Invalid low slicer PA fallback | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PETG PRESSURE_ADVANCE=-1 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.04` | Warns and falls back to material default |
| High-but-allowed slicer PA warning | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PLA PRESSURE_ADVANCE=0.30 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.04` | Accepts value and warns about high PA |
| Smooth-time upper clamp | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=ABS PRESSURE_ADVANCE=0.06 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=1.0` | Clamps to `0.2` |
| Smooth-time lower clamp | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=ABS PRESSURE_ADVANCE=0.06 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.005` | Clamps to `0.02` |
| Bed Soak parity: PCTG family | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PCTG NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `PCTG` bucket |
| Bed Soak parity: PET family alias | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PET-CF NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `PET` bucket |
| Bed Soak parity: foaming TPU alias | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=TPU-Aero NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `TPU_AIR` bucket |
| Bed Soak parity: PEBA alias | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PEBAX NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `PEBA` bucket |
| Bed Soak parity: nylon composite alias | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=PA12-CF NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `PA12` bucket |
| Bed Soak parity: support blend alias | `DYNAMIC_PRESSURE_ADVANCE MATERIAL=Support for PET-PA NOZZLE=0.4 FORCE_PA=True SMOOTH_TIME=0.04` | Maps to `NYLON` bucket |
| Compatibility wrapper pass-through | `ADAPTIVE_PRESSURE_ADVANCE MATERIAL=PETG PRESSURE_ADVANCE=0.05 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.04` | Produces same result as direct macro call |
| PRINT_START integration path | `PRINT_START MATERIAL=PLA BED_TEMP=60 EXTRUDER_TEMP=210 NOZZLE=0.4 FORCE_PA=False SMOOTH_TIME=0.04` | No unknown-parameter errors |

Tip: Use `FORCE_PA=True` while validating material bucket mapping so slicer values do not mask classification behavior.

## Notes

- Keep slicer variable names aligned with your slicer profile.
- `ADAPTIVE_PRESSURE_ADVANCE` remains available as a compatibility wrapper for existing start macros.
- `FORCE_PA=True` is useful for enforcing printer-tuned defaults.
- Tune material defaults in the module file for your machine and filament.
- Invalid slicer PA values no longer hard-stop the macro; they are handled safely with warning messages and material-aware fallback.
