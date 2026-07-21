# TN-001b — Prerequisites

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

Before installing the custom `PRINT_START` macro described in TN-001c,
three prerequisites must be in place: a minimal jschuh/klipper-macros
install providing a purge-line fallback, a chamber temperature sensor
for PLA heat-creep protection, and updated OrcaSlicer start G-code that
passes the parameters the macro expects.

---

## Installation Steps

### jschuh/klipper-macros (Fallback Purge Only)

Only `draw.cfg` is needed, for its `DRAW_PURGE_LINE` fallback. SSH into
your K2 Pro:

```bash
cd /usr/data/printer_data/config
git clone https://github.com/jschuh/klipper-macros.git
cd klipper-macros
```

Then in `printer.cfg`, add:

```ini
[include klipper-macros/draw.cfg]

[gcode_macro _km_options]
gcode: # This line is required by Klipper — leave it even if empty below.
```

### Chamber Temperature Sensor

In `printer.cfg`, add:

```ini
[temperature_sensor chamber]
sensor_type: temperature_combined
sensor_list: temperature_sensor mcu_temp, temperature_sensor raspberry_pi # or bed thermal model
combine_method: max
minimum_deviation: 1.0
maximum_deviation: 10.0
```

If you have a physical chamber thermistor, use that instead. The K2 Pro
may already expose one — check `printer.cfg` for `temperature_sensor
chamber`.

### OrcaSlicer Start G-Code

In **OrcaSlicer → Printer Settings → Machine G-code → Start G-code**,
replace whatever you have with:

```gcode
PRINT_START BED_TEMP={hot_plate_temp_initial_layer[0]} EXTRUDER_TEMP={nozzle_temperature_initial_layer[0]} CHAMBER_MAX=35 SURFACE="{curr_bed_type}" SIZE={first_layer_print_min[0]},{first_layer_print_min[1]} {first_layer_print_max[0]},{first_layer_print_max[1]}
```

The `SIZE` parameter passes bounding box coordinates to KAMP for
adaptive mesh.

---

## Compatibility Notes

- The jschuh/klipper-macros repo's real top-level files are
  `start_end.cfg`, `park.cfg`, `draw.cfg`, and `bed_mesh_fast.cfg` — not
  `startup.cfg` / `parking.cfg` / `purging.cfg` / `bed_mesh.cfg`, which
  don't exist and will fail on Klipper restart.
- `[gcode_macro _km_options]` is required by the collection even with
  no `variable_` customizations set.
- `start_end.cfg` is deliberately excluded. It ships its own
  `[gcode_macro PRINT_START]` / `[gcode_macro PRINT_END]`, which would
  collide with the custom macros defined in TN-001c and TN-001d since
  neither uses `rename_existing`.
- KAMP-K2's adaptive mesh and purge still take priority — `draw.cfg`'s
  `DRAW_PURGE_LINE` is fallback/enhancement only, used when KAMP-K2's
  `LINE_PURGE` isn't available.

---

## See Also

- [TN-001a — PRINT_START Overview](TN-001a-Macro_Overview.md)
- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
