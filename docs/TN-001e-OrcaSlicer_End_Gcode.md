# TN-001e — OrcaSlicer End G-Code

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

This technical note covers the OrcaSlicer end G-code needed to invoke
the `PRINT_END` macro defined in TN-001d.

---

## Configuration

In **OrcaSlicer → Printer Settings → Machine G-code → End G-code**:

```gcode
PRINT_END
```

---

## See Also

- [TN-001d — PRINT_END Macro](TN-001d-PRINT_END_Macro.md)
- [TN-001b — Prerequisites](TN-001b-Prerequisites.md) (Start G-code)
