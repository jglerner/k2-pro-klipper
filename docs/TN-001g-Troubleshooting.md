# TN-001g — Troubleshooting

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

This technical note collects known issues encountered when setting up
the custom `PRINT_START` macro (TN-001c), their likely causes, and
their fixes.

---

## Known Issues

| Issue | Likely Cause | Fix |
|---|---|---|
| `Unknown command: LINE_PURGE` | KAMP-K2 not installed or purge macro renamed | Check `printer.cfg` for KAMP-K2's actual purge macro name and update `PRINT_START` |
| Z-offset not applying | `curr_bed_type` not passing from Orca | Verify Start G-code syntax; check Fluidd console for passed parameters |
| Chamber temp reads 0 | No chamber sensor configured | Check `temperature_sensor chamber` in `printer.cfg`; use `mcu_temp` fallback if needed |
| Mesh takes too long | Full bed mesh instead of adaptive | Verify `SIZE` parameter is passing from Orca; check KAMP-K2 config |

---

## See Also

- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
- [TN-001b — Prerequisites](TN-001b-Prerequisites.md)
