# Bed Soak

Source file: `Bed Soak/bed_soak_module.cfg`

## Macro

- `BED_SOAK`

## Purpose

Waits for the bed to reach the resolved soak target temperature and then performs a material-aware soak to improve thermal stability before printing.

## Parameters

- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PLA+`, `PETG-CF`, `ABS`.
- `BED_TEMP=<int>`: Preferred explicit soak target temperature in C.
- `bed_target_temp=<int>`: Optional soak target alias accepted for start macro compatibility.
- `FORCE_SOAK=<bool-string>`: `True` or `False`. Forces soak when `True`.

## Behavior Summary

1. Normalizes material name into known families.
2. Determines if soak is required using material class, resolved soak temperature, or force override.
3. Waits for soak target stability with `TEMPERATURE_WAIT`.
4. Applies material-specific soak duration.
5. Emits status messages for operator visibility.

## Typical Usage

```gcode
BED_SOAK MATERIAL={material} BED_TEMP={bed_temperature} FORCE_SOAK=False
```

If the caller does not pass a soak target temperature, `BED_SOAK` falls back to the current `printer.heater_bed.target` value. This allows use in a `PRINT_START` flow where the bed target was already set earlier.

## Recommended PRINT_START Parameters

Add these parameters to your `PRINT_START` macro if you want to drive this module from slicer start G-code:

- `MATERIAL`: Filament/material name from the slicer.
- `BED_TEMP`: Preferred explicit soak target temperature.
- `FORCE_SOAK`: Manual override flag, usually `False`.

This module only uses `FORCE_SOAK`. It does not read `FORCE_PA`.

For a full `PRINT_START` example showing parameter definitions for both modules together, see [Getting Started](../getting-started.md).

Typical `PRINT_START` usage inside Klipper:

```gcode
BED_SOAK MATERIAL={material} BED_TEMP={bed_target_temp} FORCE_SOAK={force_soak}
```

If your `PRINT_START` macro already sets the bed target before calling `BED_SOAK`, you may omit `BED_TEMP` and rely on the heater target fallback:

```gcode
BED_SOAK MATERIAL={material} FORCE_SOAK={force_soak}
```

If you are using both Bed Soak and Dynamic Pressure Advance in the same `PRINT_START`, keep the force flags separate:

```gcode
DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} PRESSURE_ADVANCE={pressure_advance} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
BED_SOAK MATERIAL={material} BED_TEMP={bed_target_temp} FORCE_SOAK={force_soak}
```

## Slicer Examples

These examples show what to add in slicer start G-code so the values are available to your Klipper `PRINT_START` macro.

For this module by itself, only `FORCE_SOAK` needs to be passed. `FORCE_PA` belongs to the Dynamic Pressure Advance module and should only be added if your `PRINT_START` also calls `DYNAMIC_PRESSURE_ADVANCE`.

### PrusaSlicer

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] FORCE_SOAK=False
```

### OrcaSlicer

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] FORCE_SOAK=False
```

### Cura

Recommended slicer start G-code call:

```gcode
PRINT_START MATERIAL={material_type} BED_TEMP={material_bed_temperature_layer_0} EXTRUDER_TEMP={material_print_temperature_layer_0} FORCE_SOAK=False
```

## Duration Guidance

Approximate soak durations from current module logic:

- `PLA`, `TPU`, `PVA`: 1.5 minutes
- `PETG`, `HIPS`: 2 minutes
- `ABS`, `ASA`: 2.5 minutes
- `NYLON`, `POLY`: 3 minutes

## Debug and Testing (Before Printing)

Run these from the Klipper console (Mainsail/Fluidd) to validate parameter flow and macro behavior before launching a real print.

1. Confirm your bed target fallback path works:

```gcode
SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET=60
BED_SOAK MATERIAL=PLA FORCE_SOAK=False
```

Expected: `BED_SOAK` should resolve to the current bed target (`60`) even without `BED_TEMP`.

2. Confirm explicit temperature override is honored:

```gcode
BED_SOAK MATERIAL=PETG BED_TEMP=75 FORCE_SOAK=False
```

Expected: the macro should wait on `75` C (not the previous heater target) and apply PETG-class soak timing.

3. Confirm force behavior for quick validation cycles:

```gcode
BED_SOAK MATERIAL=PLA BED_TEMP=60 FORCE_SOAK=True
```

Expected: soak path is forced and status messages clearly show execution of the soak branch.

4. Verify your full `PRINT_START` parameter mapping without printing a part:

```gcode
PRINT_START MATERIAL=PLA BED_TEMP=60 EXTRUDER_TEMP=210 FORCE_SOAK=False
```

Expected: no unknown-parameter errors, and `BED_SOAK` receives the values your slicer is expected to pass.

Tip: For faster test loops, use low but realistic bed targets (for example `50-60` C) and watch console output for each decision branch.

## Notes

- Soak temperature resolution order is `BED_TEMP` -> `bed_target_temp` -> `printer.heater_bed.target` -> `60` C.
- Soak may be skipped when conditions do not require it.
- This macro is blocking by design while soak is running.
