# Bed Soak

Source file: `Bed Soak/bed_soak_module.cfg`

## Quick Navigation

- [Macro](#macro)
- [Purpose](#purpose)
- [Parameters](#parameters)
- [Behavior Summary](#behavior-summary)
- [Typical Usage](#typical-usage)
- [Recommended PRINT_START Parameters](#recommended-print_start-parameters)
- [Slicer Examples](#slicer-examples)
- [Duration Guidance](#duration-guidance)
- [Debug and Testing (Before Printing)](#debug-and-testing-before-printing)
- [Notes](#notes)

## Macro

- `BED_SOAK`

## Purpose

Waits for the bed to reach the resolved soak target temperature floor and then performs a material-aware soak to improve thermal stability before printing.

## Parameters

- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PLA+`, `TPU AIR`, `PEBA`, `PETG-CF`, `ABS`.
- `BED_TEMP=<int>`: Preferred explicit soak target temperature in C.
- `bed_target_temp=<int>`: Optional soak target alias accepted for start macro compatibility.
- `FORCE_SOAK=<bool-string>`: `True` or `False`. Forces soak when `True`.

Common OrcaSlicer/Bambu-style aliases normalized by this macro include `TPU-Aero`, `LW-TPU`, `PEBAX`, `PA-CF`, `PA-GF`, `PETG HF`, `Support for PLA-PETG`, and `Support for PET-PA`.

## Behavior Summary

1. Normalizes material name into known families.
2. Determines if soak is required using material class, resolved soak temperature, or force override.
3. Waits for the soak target floor with `TEMPERATURE_WAIT` (minimum threshold only).
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

- `TPU AIR`, `LW-TPU`: 1 minute
- `PLA`, `TPU`, `PVA`: 1.5 minutes
- `PETG`, `PEBA`: 2 minutes
- `PCTG`, `PET`: 2.5 minutes
- `HIPS`: 3 minutes
- `ABS`, `ASA`: 3.5 minutes
- `PP`: 4 minutes
- `PC`, `PPA`, `PPS`, `NYLON`, `PA6`, `PA11`, `PA12`, `PAHT`: 5 minutes

Reinforced `CF` and `GF` variants add 30 seconds unless they normalize to low-temp families such as `PLA`, `TPU`, `TPU AIR`, or `PVA`.

## Debug and Testing (Before Printing)

Run these from the Klipper console (Mainsail/Fluidd) to validate parameter flow and macro behavior before launching a real print.

### Quick Material Checks

Use these first when you want a fast sanity check of the material buckets and Orca-style aliases.

1. Foaming TPU aliases should map to the `TPU_AIR` bucket and only soak here because `FORCE_SOAK=True`:

```gcode
BED_SOAK MATERIAL=TPU-Aero BED_TEMP=45 FORCE_SOAK=True
BED_SOAK MATERIAL=LW-TPU BED_TEMP=45 FORCE_SOAK=True
```

Expected result: soak runs and reports about `1.0` minute.

1. PEBA aliases should map to the warm-bed elastomer bucket:

```gcode
BED_SOAK MATERIAL=PEBA BED_TEMP=70 FORCE_SOAK=False
BED_SOAK MATERIAL=PEBAX BED_TEMP=70 FORCE_SOAK=False
```

Expected result: soak runs automatically and reports about `2.0` minutes.

1. Orca/Bambu nylon composite aliases should map to the nylon family:

```gcode
BED_SOAK MATERIAL=PA-CF BED_TEMP=100 FORCE_SOAK=False
BED_SOAK MATERIAL=PA-GF BED_TEMP=90 FORCE_SOAK=False
BED_SOAK MATERIAL=PA12-CF BED_TEMP=100 FORCE_SOAK=False
```

Expected result: soak runs automatically and reports about `5.5` minutes for `PA-CF` and `PA-GF`, and about `5.5` minutes for `PA12-CF` because reinforced blends add 30 seconds.

1. Reinforced ABS/ASA aliases should keep their base family and add the composite penalty:

```gcode
BED_SOAK MATERIAL=ABS-GF BED_TEMP=100 FORCE_SOAK=False
BED_SOAK MATERIAL=ASA-CF BED_TEMP=100 FORCE_SOAK=False
```

Expected result: soak runs automatically and reports about `4.0` minutes.

1. PET-family aliases should stay in the shorter mid-temp bucket:

```gcode
BED_SOAK MATERIAL=PETG-HF BED_TEMP=75 FORCE_SOAK=False
BED_SOAK MATERIAL=PET-CF BED_TEMP=80 FORCE_SOAK=False
```

Expected result: `PETG-HF` reports about `2.0` minutes. `PET-CF` reports about `3.0` minutes because `PET` is `2.5` minutes plus 30 seconds for the reinforced blend.

Tip: watch the first `BED_SOAK active` line and the later `Bed soaking for ... minutes` line. Together they confirm alias matching, soak decision, and final duration.

### Basic Material Checks

Use these when you want to confirm the standard material families still map to the expected soak buckets.

1. Low-temp families should only soak when forced:

```gcode
BED_SOAK MATERIAL=PLA BED_TEMP=60 FORCE_SOAK=True
BED_SOAK MATERIAL=TPU BED_TEMP=50 FORCE_SOAK=True
BED_SOAK MATERIAL=PVA BED_TEMP=60 FORCE_SOAK=True
```

Expected result: each command runs the soak path and reports about `1.5` minutes.

1. Common mid-temp materials should trigger soak automatically:

```gcode
BED_SOAK MATERIAL=PETG BED_TEMP=75 FORCE_SOAK=False
BED_SOAK MATERIAL=PCTG BED_TEMP=80 FORCE_SOAK=False
BED_SOAK MATERIAL=PET BED_TEMP=80 FORCE_SOAK=False
```

Expected result: `PETG` reports about `2.0` minutes. `PCTG` and `PET` report about `2.5` minutes.

1. Utility and enclosure-prone materials should show the longer middle buckets:

```gcode
BED_SOAK MATERIAL=HIPS BED_TEMP=90 FORCE_SOAK=False
BED_SOAK MATERIAL=PP BED_TEMP=100 FORCE_SOAK=False
```

Expected result: `HIPS` reports about `3.0` minutes. `PP` reports about `4.0` minutes.

1. High-temp engineering families should all trigger the long soak bucket:

```gcode
BED_SOAK MATERIAL=PC BED_TEMP=110 FORCE_SOAK=False
BED_SOAK MATERIAL=PPA BED_TEMP=100 FORCE_SOAK=False
BED_SOAK MATERIAL=PPS BED_TEMP=110 FORCE_SOAK=False
BED_SOAK MATERIAL=NYLON BED_TEMP=90 FORCE_SOAK=False
BED_SOAK MATERIAL=PA6 BED_TEMP=90 FORCE_SOAK=False
BED_SOAK MATERIAL=PA11 BED_TEMP=90 FORCE_SOAK=False
```

Expected result: each command reports about `5.0` minutes.

1. Warp-prone structural materials should sit between mid-temp and engineering families:

```gcode
BED_SOAK MATERIAL=ABS BED_TEMP=100 FORCE_SOAK=False
BED_SOAK MATERIAL=ASA BED_TEMP=100 FORCE_SOAK=False
```

Expected result: each command reports about `3.5` minutes.

1. Confirm your bed target fallback path works:

```gcode
SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET=60
BED_SOAK MATERIAL=PLA FORCE_SOAK=False
```

Expected: `BED_SOAK` should resolve to the current bed target (`60`) even without `BED_TEMP`.

1. Confirm explicit temperature override is honored:

```gcode
BED_SOAK MATERIAL=PETG BED_TEMP=75 FORCE_SOAK=False
```

Expected: the macro should wait on `75` C (not the previous heater target) and apply PETG-class soak timing.

1. Confirm force behavior for quick validation cycles:

```gcode
BED_SOAK MATERIAL=PLA BED_TEMP=60 FORCE_SOAK=True
```

Expected: soak path is forced and status messages clearly show execution of the soak branch.

1. Verify your full `PRINT_START` parameter mapping without printing a part:

```gcode
PRINT_START MATERIAL=PLA BED_TEMP=60 EXTRUDER_TEMP=210 FORCE_SOAK=False
```

Expected: no unknown-parameter errors, and `BED_SOAK` receives the values your slicer is expected to pass.

Tip: For faster test loops, use low but realistic bed targets (for example `50-60` C) and watch console output for each decision branch.

## Notes

- Soak temperature resolution order is `BED_TEMP` -> `bed_target_temp` -> `printer.heater_bed.target` -> `60` C.
- Soak may be skipped when conditions do not require it.
- This macro is blocking by design while soak is running. Console commands entered during soak are queued and execute after the macro returns.
- The wait step uses only `MINIMUM` (no `MAXIMUM`) to reduce long stalls from normal PID overshoot.
