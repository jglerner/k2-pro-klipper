# TN-001c — PRINT_START Macro

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

This technical note contains the full custom `PRINT_START` macro for
the Creality K2 Pro, combining chamber safety checks, bed-surface
Z-offset detection, KAMP-K2 adaptive bed meshing, and an adaptive purge
sequence. The prerequisites in TN-001b must be completed first.

---

## Macro Definition

Add this to `printer.cfg` (or a dedicated `macros.cfg` if you prefer):

```ini
[gcode_macro PRINT_START]
gcode:
    # ============================================================
    # PARAMETERS FROM ORCASLICER
    # ============================================================
    {% set BED_TEMP = params.BED_TEMP|default(60)|int %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|int %}
    {% set CHAMBER_MAX = params.CHAMBER_MAX|default(35)|int %}
    {% set SURFACE = params.SURFACE|default('Smooth PEI Plate')|string %}
    {% set SIZE = params.SIZE|default('0,0,0,0')|string %}

    # Parse SIZE into min/max coordinates
    {% set coords = SIZE.split(',') %}
    {% set min_x = coords[0]|float %}
    {% set min_y = coords[1]|float %}
    {% set max_x = coords[2]|float %}
    {% set max_y = coords[3]|float %}

    # ============================================================
    # 1. SAFETY & SANITY CHECKS
    # ============================================================
    # Check chamber temp (PLA safety)
    {% set chamber_temp = printer['temperature_sensor chamber'].temperature|default(20)|float %}
    {% if chamber_temp > CHAMBER_MAX %}
        {action_respond_info("WARNING: Chamber temp %.1fC exceeds PLA safe limit (%dC). Cracking door or waiting..." % (chamber_temp, CHAMBER_MAX))}
        # Optional: pause and wait for cooldown
        # TEMPERATURE_WAIT SENSOR="temperature_sensor chamber" MAXIMUM={CHAMBER_MAX}
    {% endif %}

    # Check filament sensor (if installed)
    {% if 'filament_switch_sensor runout' in printer %}
        {% if not printer['filament_switch_sensor runout'].filament_detected %}
            {action_raise_error("FILAMENT RUNOUT: Load filament before starting print.")}
        {% endif %}
    {% endif %}

    # ============================================================
    # 2. HEATING SEQUENCE (Optimized for PLA)
    # ============================================================
    # Home if not already homed
    {% if 'x' not in printer.toolhead.homed_axes or 'y' not in printer.toolhead.homed_axes or 'z' not in printer.toolhead.homed_axes %}
        G28
    {% endif %}

    # Move to park position while heating (prevents nozzle oozing on bed)
    G0 X150 Y10 Z50 F12000

    # Heat bed first (PLA doesn't need aggressive soak, but brief preheat helps)
    M140 S{BED_TEMP}
    M190 S{BED_TEMP}

    # Brief bed soak for PLA (5 seconds is plenty — adjust if your room is cold)
    G4 P5000

    # Heat hotend
    M104 S{EXTRUDER_TEMP}
    M109 S{EXTRUDER_TEMP}

    # ============================================================
    # 3. BED SURFACE & Z-OFFSET
    # ============================================================
    # Apply Z-offset based on bed surface type
    {% set z_offset_adjustment = 0.0 %}
    {% if 'Textured' in SURFACE %}
        {% set z_offset_adjustment = -0.04 %}
        {action_respond_info("Bed surface: Textured PEI — applying Z-offset %.3f" % z_offset_adjustment)}
    {% elif 'Smooth' in SURFACE %}
        {% set z_offset_adjustment = 0.0 %}
        {action_respond_info("Bed surface: Smooth PEI — no Z-offset adjustment")}
    {% elif 'High Temp' in SURFACE %}
        {% set z_offset_adjustment = 0.06 %}
        {action_respond_info("Bed surface: High Temp Plate — applying Z-offset +%.3f" % z_offset_adjustment)}
    {% else %}
        {action_respond_info("Bed surface: '%s' not recognized, using default Z-offset" % SURFACE)}
    {% endif %}
    SET_GCODE_OFFSET Z={z_offset_adjustment} MOVE=1

    # ============================================================
    # 4. ADAPTIVE BED MESH (KAMP-K2)
    # ============================================================
    # Use KAMP's adaptive mesh if bounding box is valid
    {% if min_x != max_x and min_y != max_y %}
        {action_respond_info("Running KAMP adaptive mesh for bounding box: X%.1f-%.1f Y%.1f-%.1f" % (min_x, max_x, min_y, max_y))}
        # KAMP-K2 adaptive mesh macro (from grant0013's repo)
        # This should already exist in your config
        BED_MESH_CALIBRATE mesh_min={min_x},{min_y} mesh_max={max_x},{max_y}
    {% else %}
        {action_respond_info("No valid print size detected — falling back to full bed mesh")}
        BED_MESH_CALIBRATE
    {% endif %}

    # ============================================================
    # 5. ADAPTIVE PURGE (KAMP-K2)
    # ============================================================
    # Let KAMP-K2 handle the adaptive purge.
    # KAMP-K2's (and upstream KAMP's) purge macro is named LINE_PURGE, not
    # ADAPTIVE_PURGE — that name doesn't exist and the check would silently
    # always fall through to the jschuh fallback below.
    {% if 'LINE_PURGE' in printer.gcode.commands %}
        LINE_PURGE
    {% else %}
        # Fallback: jschuh's smart purge line (from draw.cfg)
        {% if 'DRAW_PURGE_LINE' in printer.gcode.commands %}
            DRAW_PURGE_LINE
        {% else %}
            # Ultimate fallback: simple purge line
            G0 X10 Y10 Z0.2 F6000
            G1 X60 Y10 E15 F1500
            G1 X60 Y15 E0.5
            G1 X10 Y15 E15 F1500
            G0 Z2
        {% endif %}
    {% endif %}

    # ============================================================
    # 6. FINAL PREP
    # ============================================================
    # Reset extruder
    G92 E0

    # Move to start position
    G0 X{min_x + 10} Y{min_y + 10} Z0.2 F12000

    # Enable part cooling fan (PLA needs it from layer 1)
    M106 S255

    {action_respond_info("PRINT_START complete — commencing print")}
```

---

## Compatibility Notes

- This macro is safe to define directly (no `rename_existing`) only
  because TN-001b deliberately excludes jschuh's `start_end.cfg`, which
  would otherwise define its own conflicting `[gcode_macro PRINT_START]`.
- The `BED_MESH_CALIBRATE mesh_min=... mesh_max=...` call is correct
  for K2 Pro. Creality's stock firmware hijacks a bare
  `BED_MESH_CALIBRATE` call and crashes on `MESH_MIN`/`MESH_MAX`, but
  KAMP-K2 patches this to require those params — exactly what this
  macro passes.

---

## See Also

- [TN-001b — Prerequisites](TN-001b-Prerequisites.md)
- [TN-001d — PRINT_END Macro](TN-001d-PRINT_END_Macro.md)
- [TN-001g — Troubleshooting](TN-001g-Troubleshooting.md)
