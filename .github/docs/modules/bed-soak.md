# Bed Soak

Source file: `Bed Soak/bed_soak_module.cfg`

## Macro

- `BED_SOAK`

## Purpose

Waits for the bed to reach target temperature and then performs a material-aware soak to improve thermal stability before printing.

## Parameters

- `MATERIAL=<string>`: Material profile name, for example `PLA`, `PLA+`, `PETG-CF`, `ABS`.
- `BED_TEMP=<int>`: Target bed temperature in C.
- `FORCE=<bool-string>`: `True` or `False`. Forces soak when `True`.

## Behavior Summary

1. Normalizes material name into known families.
2. Determines if soak is required using material class, bed temperature, or force override.
3. Waits for bed temperature stability with `TEMPERATURE_WAIT`.
4. Applies material-specific soak duration.
5. Emits status messages for operator visibility.

## Typical Usage

```gcode
BED_SOAK MATERIAL={material} BED_TEMP={bed_temperature} FORCE=False
```

## Duration Guidance

Approximate soak durations from current module logic:

- `PLA`, `TPU`, `PVA`: 1.5 minutes
- `PETG`, `HIPS`: 2 minutes
- `ABS`, `ASA`: 2.5 minutes
- `NYLON`, `POLY`: 3 minutes

## Notes

- If `BED_TEMP <= 0`, the macro falls back to `60` C.
- Soak may be skipped when conditions do not require it.
- This macro is blocking by design while soak is running.
