# Caselight Effects Technical Guide

Version: 1.0
Last Updated: 2026-07-12

## 1. Basic Chaining Syntax

Simple chain, run macros one after another:

```cfg
MACRO1
MACRO2
MACRO3
```

With delays between effects:

```cfg
MACRO1
G4 P<milliseconds>
MACRO2
G4 P<milliseconds>
MACRO3
```

Template: Morning/Startup

```cfg
[gcode_macro STARTUP_LIGHTS]
description: Morning/Startup lighting sequence
gcode:
  ; Gentle wake-up
  PULSE_CASELIGHT PULSES=3 DURATION=2000
  G4 P1000
  FADE_CASELIGHT PULSES=1
  G4 P1000
  WAVE_SMOOTH_CASELIGHT PULSES=5
  SET_PIN PIN=caselight VALUE=1
```

Template: Evening/Wind Down

```cfg
[gcode_macro WIND_DOWN_LIGHTS]
description: Evening wind-down lighting sequence
gcode:
  ; Slow calming effect
  PULSE_CASELIGHT PULSES=5 DURATION=3000
  G4 P2000
  FADE_CASELIGHT PULSES=1
  G4 P2000
  WAVE_SMOOTH_CASELIGHT PULSES=3
  G4 P2000
  ; End at dim for night
  SET_PIN PIN=caselight VALUE=0.2
```

## 2. Quick Command Sequences (Console Only)

Quick chain, copy and paste these into console.

Basic chain with pauses:

```cfg
HEARTBEAT_CASELIGHT BEATS=3
G4 P500
PULSE_CASELIGHT PULSES=2 DURATION=1000
G4 P500
WAVE_SMOOTH_CASELIGHT PULSES=3
```

Double heartbeat alert:

```cfg
HEARTBEAT_CASELIGHT BEATS=5
G4 P300
HEARTBEAT_CASELIGHT BEATS=5
```

Cinematic sequence:

```cfg
FADE_CASELIGHT PULSES=1
G4 P500
PULSE_CASELIGHT PULSES=3 DURATION=1500
G4 P500
FADE_CASELIGHT PULSES=1
```

Celebration sequence:

```cfg
WAVE_CASELIGHT PULSES=3
G4 P500
HEARTBEAT_CASELIGHT BEATS=3
G4 P500
WAVE_SMOOTH_CASELIGHT PULSES=3
```

Error sequence (repeating):

```cfg
HEARTBEAT_CASELIGHT BEATS=3
G4 P200
HEARTBEAT_CASELIGHT BEATS=3
G4 P200
HEARTBEAT_CASELIGHT BEATS=3
```

## 3. Advanced Chaining with Loops

Loop-based sequences (for console use).

Run multiple cycles of the same pattern. This runs three cycles of heartbeat plus pause:

```cfg
{% for i in range(3) %}
  HEARTBEAT_CASELIGHT BEATS=3
  G4 P500
{% endfor %}
```

Alternating patterns:

```cfg
{% for i in range(3) %}
  WAVE_CASELIGHT PULSES=2
  G4 P500
  PULSE_CASELIGHT PULSES=2 DURATION=500
  G4 P500
{% endfor %}
```

Escalating pattern:

```cfg
{% for i in range(1, 4) %}
  HEARTBEAT_CASELIGHT BEATS={i}
  G4 P500
{% endfor %}
```

## 4. Conditional Sequences

Example: different sequences based on state.

```cfg
[gcode_macro SMART_LIGHTS]
description: Smart lighting based on printer state
gcode:
  {% set state = params.STATE|default('idle') %}

  {% if state == 'printing' %}
    WAVE_SMOOTH_CASELIGHT PULSES=5
    PULSE_CASELIGHT PULSES=3 DURATION=1500
  {% elif state == 'complete' %}
    HEARTBEAT_CASELIGHT BEATS=5
    WAVE_CASELIGHT PULSES=3
    G4 P500
    HEARTBEAT_CASELIGHT BEATS=3
  {% elif state == 'error' %}
    HEARTBEAT_CASELIGHT BEATS=3
    G4 P300
    HEARTBEAT_CASELIGHT BEATS=3
    G4 P300
    HEARTBEAT_CASELIGHT BEATS=3
  {% else %}
    PULSE_CASELIGHT PULSES=2 DURATION=1000
  {% endif %}

  SET_PIN PIN=caselight VALUE=1
```

## 5. Timing Reference

Common timing values in milliseconds:

- G4 P100 = 0.1 seconds (very fast)
- G4 P250 = 0.25 seconds (quick flash)
- G4 P500 = 0.5 seconds (standard pause)
- G4 P1000 = 1.0 seconds (long pause)
- G4 P2000 = 2.0 seconds (very long pause)
- G4 P5000 = 5.0 seconds (extended pause)

Recommended pause between effects:

- Between heartbeats: G4 P300-500
- Between waves: G4 P500-1000
- Between pulses: G4 P500-1000
- After fades: G4 P1000-2000
- For dramatic effect: G4 P2000-3000

## 6. Sequence Duration Estimates

Single effects:

- HEARTBEAT_CASELIGHT BEATS=3 -> ~1.5 seconds
- FADE_CASELIGHT PULSES=1 -> ~3.4 seconds
- WAVE_CASELIGHT PULSES=3 -> ~3.0 seconds
- PULSE_CASELIGHT PULSES=3 -> ~3.0 seconds (at 1000 ms each)

Combined sequences:

- Basic chain (three effects) -> ~8-10 seconds
- Celebration sequence -> ~10-12 seconds
- Alert sequence -> ~6-8 seconds
- Ambient mode (short) -> ~20-30 seconds
- Ambient mode (long) -> ~2-5 minutes

## 7. Sequence Design Tips

1. Start with an attention-grabbing effect (HEARTBEAT or FADE).
2. Follow with smooth transitions (WAVE_SMOOTH or PULSE).
3. End with a conclusive effect (single HEARTBEAT or restore to full).
4. For notifications, keep under five seconds total.
5. For alerts, keep under ten seconds and repeat if needed.
6. For ambiance, loops can run indefinitely.

Example pattern:

```text
GRAB ATTENTION -> HOLD INTEREST -> CONCLUDE
HEARTBEAT      -> WAVE_SMOOTH   -> SET_PIN VALUE=1
FADE           -> PULSE         -> HEARTBEAT BEATS=1
```

## 8. Troubleshooting Chains

If a sequence stops unexpectedly:

1. Check for errors in console.
2. Ensure each macro completes before the next starts.
3. Add G4 delays between effects.
4. Try fewer effects first.

If sequence is too fast:

1. Increase G4 delays between effects.
2. Use the DURATION parameter on PULSE_CASELIGHT.
3. Add extra pauses with G4.

If sequence is too slow:

1. Decrease G4 delays.
2. Use fewer effects.
3. Use faster macros (WAVE instead of WAVE_SMOOTH).

To stop a running sequence:

```cfg
SET_PIN PIN=caselight VALUE=1
```