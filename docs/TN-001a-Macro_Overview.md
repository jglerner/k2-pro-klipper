# TN-001a — PRINT_START Overview

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

This technical note introduces the objectives and architecture of a custom
`PRINT_START` macro developed for the Creality K2 Pro.

The macro extends an existing KAMP-K2 installation while preserving its
original functionality. Rather than replacing established components,
it adds several engineering improvements that simplify daily printing
and improve reliability.

---

## Functional Overview

The macro performs the following functions during print initialization,
in approximately this order:

- Perform pre-print sanity checks.
- Monitor chamber temperature for PLA printing.
- Detect the active build surface and apply the appropriate Z-offset.
- Generate an adaptive bed mesh using the print bounding box.
- Execute an adaptive purge sequence before printing begins.

Each enhancement is intentionally modular and can be enabled,
modified, or removed independently.

---

## Engineering Intent

The design goals are:

- improve printing reliability;
- reduce unnecessary probing time;
- minimize user intervention;
- preserve compatibility with KAMP-K2;
- keep all enhancements easy to understand and maintain.

---

## See Also

- [TN-001b — Prerequisites](TN-001b-Prerequisites.md)
- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
