# Adaptive Pressure Advance

Source file: `Adapative Pressure Advance/adaptive_pressure_advance_module.cfg`

## Macro

- `ADAPTIVE_PRESSURE_ADVANCE`

## Purpose

Selects a pressure advance value using slicer input, material defaults, nozzle scaling, and a force override.

## Parameters

- `PRESSURE_ADVANCE=<float>`: Optional slicer-provided pressure advance.
- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PETG-CF`, `ABS`.
- `NOZZLE=<float>`: Nozzle diameter in mm. Default is `0.4`.
- `FORCE=<bool-string>`: `True` or `False`. When `True`, ignores slicer pressure advance and uses module defaults.
- `SMOOTH_TIME=<float>`: Pressure advance smooth time. Default is `0.040`.

## Behavior Summary

1. Normalizes material family from `MATERIAL`.
2. Selects base default pressure advance by material.
3. Applies nozzle-based scaling factor.
4. Chooses slicer value unless `FORCE=True`.
5. Clamps pressure advance to `0.0` through `0.5`.
6. Clamps smooth time to a safe range (`0.040` fallback if invalid, max `0.2`).
7. Applies values via `SET_PRESSURE_ADVANCE`.

## Start G-code Example

```gcode
ADAPTIVE_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE=[force_pa] SMOOTH_TIME=[smooth_time]
```

If your slicer does not expose a smooth time variable:

```gcode
ADAPTIVE_PRESSURE_ADVANCE MATERIAL=[filament_type] PRESSURE_ADVANCE=[pressure_advance] NOZZLE=[nozzle_diameter] FORCE=[force_pa] SMOOTH_TIME=0.040
```

## Notes

- Keep slicer variable names aligned with your slicer profile.
- `FORCE=True` is useful for enforcing printer-tuned defaults.
- Tune material defaults in the module file for your machine and filament.
