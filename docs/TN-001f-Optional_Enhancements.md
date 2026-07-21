# TN-001f — Optional Enhancements

> **Printer:** Creality K2 Pro
>
> **Firmware:** 1.1.6.3
>
> **Slicer:** OrcaSlicer
>
> **Interface:** Fluidd
>
> **Mods:** KAMP-K2 (grant0013), K2 Camera
>
> **Environment:** Bedroom, PLA-only, fume-sensitive
>
> **Status:** Draft
>
> **Part of:** TN-001 — PLA Bedroom PRINT_START

---

## Summary

This technical note collects optional enhancements that build on the
core `PRINT_START`/`PRINT_END` macros (TN-001c, TN-001d) but aren't
required for a working setup: chamber exhaust fan control, a timelapse
trigger, and verifying exclude-object support.

---

## Enhancements

### Chamber Exhaust Fan Control

If you add a low-RPM exhaust fan (e.g., Noctua 40mm) to vent warm air:

```ini
[fan_generic chamber_exhaust]
pin: PA8 # Check your board pinout
max_power: 0.4 # Run at 40% max to keep it quiet
```

Then in `PRINT_START`, after heating:

```gcode
# Turn on exhaust fan if chamber > 30C
{% if chamber_temp > 30 %}
    SET_FAN_SPEED FAN=chamber_exhaust SPEED=0.3
{% endif %}
```

### Timelapse Trigger

If you install **Moonraker Timelapse**:

```gcode
# In PRINT_START, after G28:
TIMELAPSE_TAKE_FRAME
```

And in `PRINT_END`:

```gcode
TIMELAPSE_RENDER
```

### Exclude Object (Already in KAMP-K2?)

Verify it's enabled in `printer.cfg`:

```ini
[exclude_object]
```

In OrcaSlicer: **Process → Others → Label objects** → Enable

---

## See Also

- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
- [TN-001d — PRINT_END Macro](TN-001d-PRINT_END_Macro.md)
