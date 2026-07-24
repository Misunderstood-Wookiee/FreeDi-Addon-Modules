# Getting Started

## Prerequisites

Before using any module in this repository, confirm all of the following:

1. Your printer is flashed with FreeDi Firmware.
2. Your printer config is based on FreeDi base machine configuration files.
3. Klipper can see these module files via your include paths.

## Installation

1. Copy this repository into your Klipper config storage location.
2. Include the module file(s) you want in your main printer configuration.
3. Update you macros with the module parameters when necessary.
4. Restart Klipper after adding includes.
5. Test each macro manually before wiring it into slicer automation.

## Recommended Validation Flow

1. Run each macro once from the console with safe/default parameters.
2. Confirm there are no unknown command errors.
3. Confirm expected `RESPOND` and display messages appear.
4. Add macros to slicer start G-code only after manual verification.

## Slicer Integration Notes

- Pass string values for booleans as `True` or `False` where required.
- Keep parameter names exact (for example `SMOOTH_TIME`, not `SMOOTHTIME`).
- If slicer variables are unavailable, use fixed literals in G-code.

## Combined PRINT_START Example

If you are using both Dynamic Pressure Advance and Bed Soak, your `PRINT_START` macro should explicitly define the parameters those modules expect before calling them.

Example `PRINT_START` macro fragment:

```gcode
[gcode_macro PRINT_START]
gcode:
    # Set bed soak and dynamic pressure advance variables
    {% set material = params.MATERIAL|default("PLA")|string %}
    {% set bed_target_temp = params.BED_TEMP|default(60)|int %}
    {% set pa = params.PRESSURE_ADVANCE|default(None) %}
    {% set nozzle = params.NOZZLE|default(0.4)|float %}
    {% set force_soak = params.FORCE_SOAK|default("False")|lower == "true" %}
    {% set force_pa = params.FORCE_PA|default("False")|lower == "true" %}
    {% set smooth_time = params.SMOOTH_TIME|default(0.03)|float %}

    BED_SOAK MATERIAL={material} BED_TEMP={bed_target_temp} FORCE_SOAK={force_soak}

    {% if pa is not none %}
        DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} PRESSURE_ADVANCE={pa} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
    {% else %}
        DYNAMIC_PRESSURE_ADVANCE MATERIAL={material} NOZZLE={nozzle} FORCE_PA={force_pa} SMOOTH_TIME={smooth_time}
    {% endif %}
```

This example keeps the force flags separate:

- `FORCE_SOAK` is only used by `BED_SOAK`.
- `FORCE_PA` is only used by `DYNAMIC_PRESSURE_ADVANCE`.

If your printer config already sets the bed target before `BED_SOAK`, you may omit `BED_TEMP` from the `BED_SOAK` call and rely on `printer.heater_bed.target` as the module fallback.

## Slicer to PRINT_START Examples

These examples show what the slicer should pass into `PRINT_START` when both modules are enabled.

### PrusaSlicer

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] FORCE_PA=False FORCE_SOAK=False SMOOTH_TIME=0.03
```

If you maintain a slicer-side pressure advance variable in your profile:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False FORCE_SOAK=False SMOOTH_TIME=0.03
```

### OrcaSlicer

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] FORCE_PA=False FORCE_SOAK=False SMOOTH_TIME=0.03
```

If you maintain a slicer-side pressure advance variable in your profile:

```gcode
PRINT_START MATERIAL=[filament_type] BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[nozzle_temperature_initial_layer] NOZZLE=[nozzle_diameter] PRESSURE_ADVANCE=[pressure_advance] FORCE_PA=False FORCE_SOAK=False SMOOTH_TIME=0.03
```

### Cura

```gcode
PRINT_START MATERIAL={material_type} BED_TEMP={material_bed_temperature_layer_0} EXTRUDER_TEMP={material_print_temperature_layer_0} NOZZLE={machine_nozzle_size} FORCE_PA=False FORCE_SOAK=False SMOOTH_TIME=0.03
```

In Cura, pressure advance is typically better left to the Klipper module unless you already maintain a custom variable for it.

## Documentation Conventions

- Parameter names use uppercase to match macro usage.
- Times are listed in milliseconds for `G4 P...` contexts.
- Example snippets are intended to be copy/paste friendly.
