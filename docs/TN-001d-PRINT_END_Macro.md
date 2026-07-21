# TN-001d — PRINT_END Macro

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

This technical note contains the matching `PRINT_END` macro for clean,
safe print completion: heaters off, filament retracted, toolhead
parked, motors disabled. It pairs with the `PRINT_START` macro in
TN-001c.

---

## Macro Definition

```ini
[gcode_macro PRINT_END]
gcode:
    # Turn off heaters
    M104 S0
    M140 S0

    # Retract filament slightly to prevent oozing
    G91
    G1 E-2 F1800
    G90

    # Move nozzle away from print
    G0 X150 Y280 Z{printer.toolhead.position.z + 10} F12000

    # Turn off part cooling fan
    M106 S0

    # Disable motors
    M84

    # Park toolhead
    G0 X150 Y280 F12000

    {action_respond_info("PRINT_END complete — print finished")}
```

---

## Compatibility Notes

- Same caveat as TN-001c — this is safe to define directly only
  because TN-001b excludes jschuh's `start_end.cfg`, which ships its
  own `[gcode_macro PRINT_END]`.

---

## See Also

- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
- [TN-001e — OrcaSlicer End G-Code](TN-001e-OrcaSlicer_End_Gcode.md)
