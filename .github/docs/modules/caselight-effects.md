# Caselight Effects

Source files:

- `Caselight Effects/caselight_effects_core_module.cfg`
- `Caselight Effects/caselight_effects_sequences_module.cfg`

## Dependency Order

The sequences module depends on the core module.

Load order requirement:

1. Include `caselight_effects_core_module.cfg` first.
2. Include `caselight_effects_sequences_module.cfg` second.

## Core Effect Macros

- `HEARTBEAT_CASELIGHT`
- `FADE_CASELIGHT`
- `WAVE_CASELIGHT`
- `WAVE_SMOOTH_CASELIGHT`
- `PULSE_CASELIGHT`

These are reusable building blocks that can be called directly or chained in custom sequences.

## Predefined Sequence Macros

- `PRINT_START_LIGHTS`
- `PRINT_DONE_CELEBRATION`
- `ERROR_ALERT`
- `NOTIFICATION_SUBTLE`
- `ATTENTION_NEEDED`
- `AMBIENT_CALM`
- `EMERGENCY_STOP_LIGHTS`
- `PREHEAT_DONE`
- `AMBIENT_LONG`
- `STARTUP_LIGHTS`
- `WIND_DOWN_LIGHTS`

## Quick Usage Examples

```gcode
PRINT_START_LIGHTS
ERROR_ALERT
AMBIENT_LONG
```

## Building Custom Sequences

Use core effects to build your own high-level macro:

```gcode
[gcode_macro MY_CUSTOM_LIGHT_SEQUENCE]
description: Custom lighting sequence
gcode:
  HEARTBEAT_CASELIGHT BEATS=2
  G4 P300
  WAVE_SMOOTH_CASELIGHT PULSES=3
  G4 P300
  PULSE_CASELIGHT PULSES=2 DURATION=1200
  SET_PIN PIN=caselight VALUE=1
```

## Notes

- Most effects are blocking while they run.
- Keep pulse counts conservative for long prints.
- End custom sequences with a known brightness state.
