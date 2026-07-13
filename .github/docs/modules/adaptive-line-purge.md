# Adaptive Line Purge

Source file: `Adapative Line Purge/adaptive_line_purge.cfg`

## Macro

- `LINE_PURGE`

## Purpose

Places a purge line near the active print area using `exclude_object` polygon data when available, while preserving printer state before returning control.

## Config Variables

- `variable_purge_height=<float>`: Z height for purge motion. Default is `0.8`.
- `variable_tip_distance=<float>`: Filament tip move before purge extrusion. Default is `-4.0`.
- `variable_purge_margin=<int>`: Margin from print area edge used for purge placement. Default is `15`.
- `variable_purge_amount=<int>`: Purge line extrusion distance. Default is `50`.
- `variable_flow_rate=<float>`: Purge flow in mm3/s. Default is `15`.

## Behavior Summary

1. Reads object polygons from `exclude_object`.
2. Calculates candidate purge origins and centers from object bounds.
3. Chooses purge direction based on available space.
4. Uses firmware retraction (`G10`/`G11`) when configured, otherwise falls back to explicit retract moves.
5. Saves and restores G-code state around the purge routine.
6. Skips purge and emits an info message if `max_extrude_cross_section` is below safe threshold.

## Typical Usage

```gcode
LINE_PURGE
```

## Notes

- Best results require slicer output with `exclude_object` data enabled.
- If no objects are present, placement defaults near the machine origin.
- This macro is blocking while purge moves and extrusion are running.
