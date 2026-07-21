# TN-001h — Quick Reference

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

This technical note is a quick-reference table of every mod involved
in the PLA Bedroom PRINT_START setup, what each one does, and its
current install status.

---

## Mod Status

| Mod | What It Does | Your Status |
|---|---|---|
| **KAMP-K2** | Adaptive mesh + purge per print size | ✅ Installed |
| **K2 Camera** | Better camera support | ✅ Installed |
| **jschuh macros** | Purge-line fallback | Add for this macro |
| **Chamber temp sensor** | PLA heat creep prevention | Add to `printer.cfg` |
| **Custom PRINT_START** | Combines everything | Add this macro |
| **Exclude Object** | Cancel single failed parts | Verify enabled |

---

## Metadata

- **Generated:** 2026-07-18
- **For:** Creality K2 Pro, FW 1.1.6.3, PLA-only bedroom setup
- **Next step:** SSH into printer, verify KAMP macro names, paste this
  `PRINT_START`, test with a small calibration cube.

---

## See Also

- [TN-001a — PRINT_START Overview](TN-001a-Macro_Overview.md)
- [TN-001g — Troubleshooting](TN-001g-Troubleshooting.md)
