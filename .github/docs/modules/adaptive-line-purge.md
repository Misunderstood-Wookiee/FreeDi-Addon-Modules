# Adaptive Line Purge

Source file: `Adapative Line Purge/adaptive_line_purge.module.cfg`

## Quick Navigation

- [Macro](#macro)
- [Purpose](#purpose)
- [Parameters](#parameters)
- [Behavior Summary](#behavior-summary)
- [Typical Usage](#typical-usage)
- [Notes](#notes)

## Macro

- `LINE_PURGE`

## Purpose

Places a purge line near the active print area using `exclude_object` polygon data when available, while preserving printer state before returning control. The macro now also avoids unsafe bed-edge moves, skips purge when no safe line space remains, and can optionally use a zig-zag purge path instead of a straight line.

## Parameters

- `variable_purge_height=<float>`: Z height for purge motion. Default is `0.8`.
- `variable_tip_distance=<float>`: Filament tip move before purge extrusion. Default is `-4.0`.
- `variable_purge_margin=<int>`: Margin from print area edge used for purge placement. Default is `15`.
- `variable_purge_amount=<int>`: Purge line extrusion distance. Default is `50`.
- `variable_flow_rate=<float>`: Purge flow in mm3/s. Default is `15`.
- `variable_purge_pattern=<string>`: Purge path style. Use `straight` for the default straight-line purge, or `zigzag` for an alternating wave-like path.
- `variable_purge_wave_amplitude=<float>`: Maximum offset from the centerline when using the zig-zag pattern. Smaller values give a softer wave; larger values make the path more pronounced.
- `variable_purge_wave_spacing=<float>`: Distance between wave points when using the zig-zag pattern. Smaller values create a smoother wave; larger values create a sharper zig-zag.

## Behavior Summary

1. Reads object polygons from `exclude_object` when available and falls back safely if they are missing.
2. Calculates candidate purge origins and centers from object bounds, while clamping positions to the printable bed area.
3. Chooses purge direction based on available space and skips the purge if no safe line space remains.
4. Uses firmware retraction (`G10`/`G11`) when configured, otherwise falls back to explicit retract moves.
5. Saves and restores G-code state around the purge routine.
6. Skips purge and emits an info message if `max_extrude_cross_section` is below safe threshold.
7. Supports an optional zig-zag path when `variable_purge_pattern` is set to `zigzag`.

## Typical Usage

```gcode
LINE_PURGE
```

## Notes

- Best results require slicer output with `exclude_object` data enabled.
- If no objects are present, placement defaults near the machine origin.
- The default behavior is a straight-line purge; enable `variable_purge_pattern="zigzag"` for a wave-like path.
- This macro is blocking while purge moves and extrusion are running.
