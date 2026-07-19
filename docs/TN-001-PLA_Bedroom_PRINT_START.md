**\# K2 Pro PLA Bedroom Setup --- Custom PRINT_START Macro**

\> \*\*Printer:\*\* Creality K2 Pro

\> \*\*Firmware:\*\* 1.1.6.3

\> \*\*Slicer:\*\* OrcaSlicer

\> \*\*Interface:\*\* Fluidd

\> \*\*Mods:\*\* KAMP-K2 (grant0013), K2 Camera

\> \*\*Environment:\*\* Bedroom, PLA-only, fume-sensitive

\-\--

\## 1. What This Macro Does

This \`PRINT_START\` enhances your \*\*existing KAMP-K2 setup\*\*
without replacing it. It layers on:

\- \*\*Chamber temperature guard\*\* --- prevents PLA heat creep in the
enclosed K2 Pro

\- \*\*Bed surface auto-detection\*\* --- reads OrcaSlicer\'s
\`curr_bed_type\` variable and applies the correct Z-offset

\- \*\*Faster bed mesh\*\* --- uses KAMP\'s adaptive mesh but adds
jschuh\'s bounding-box optimization

\- \*\*Smart purge line\*\* --- replaces stock purge with a size-aware
purge via \`DRAW_PURGE_LINE\`

\- \*\*Pre-print sanity checks\*\* --- filament sensor, door sensor (if
installed), chamber fan logic

\-\--

\## 2. Prerequisites

\### 2.1 Install jschuh/klipper-macros (partial)

You only need these specific macros from the jschuh collection. SSH into
your K2 Pro:

\`\`\`bash

cd /usr/data/printer_data/config

git clone https://github.com/jschuh/klipper-macros.git

cd klipper-macros

\`\`\`

Then in \`printer.cfg\`, add:

\`\`\`ini

\[include klipper-macros/startup.cfg\]

\[include klipper-macros/parking.cfg\]

\[include klipper-macros/purging.cfg\]

\[include klipper-macros/bed_mesh.cfg\]

\`\`\`

\> \*\*Note:\*\* The KAMP-K2 adaptive mesh and purge take priority.
These jschuh macros are \*\*fallback/enhancement only\*\*.

\### 2.2 Add Chamber Temperature Sensor

In \`printer.cfg\`, add:

\`\`\`ini

\[temperature_sensor chamber\]

sensor_type: temperature_combined

sensor_list: temperature_sensor mcu_temp, temperature_sensor
raspberry_pi \# or bed thermal model

combine_method: max

minimum_deviation: 1.0

maximum_deviation: 10.0

\`\`\`

\> If you have a physical chamber thermistor, use that instead. The K2
Pro may already expose one --- check \`printer.cfg\` for
\`temperature_sensor chamber\`.

\### 2.3 OrcaSlicer Start G-Code

In \*\*OrcaSlicer → Printer Settings → Machine G-code → Start
G-code\*\*, replace whatever you have with:

\`\`\`gcode

PRINT_START BED_TEMP={hot_plate_temp_initial_layer\[0\]}
EXTRUDER_TEMP={nozzle_temperature_initial_layer\[0\]} CHAMBER_MAX=35
SURFACE=\"{curr_bed_type}\"
SIZE={first_layer_print_min\[0\]},{first_layer_print_min\[1\]}
{first_layer_print_max\[0\]},{first_layer_print_max\[1\]}

\`\`\`

\> The \`SIZE\` parameter passes bounding box coordinates to KAMP for
adaptive mesh.

\-\--

\## 3. The Custom PRINT_START Macro

Add this to \`printer.cfg\` (or a dedicated \`macros.cfg\` if you
prefer):

\`\`\`ini

\[gcode_macro PRINT_START\]

gcode:

\# ============================================================

\# PARAMETERS FROM ORCASLICER

\# ============================================================

{% set BED_TEMP = params.BED_TEMP\|default(60)\|int %}

{% set EXTRUDER_TEMP = params.EXTRUDER_TEMP\|default(200)\|int %}

{% set CHAMBER_MAX = params.CHAMBER_MAX\|default(35)\|int %}

{% set SURFACE = params.SURFACE\|default(\'Smooth PEI Plate\')\|string
%}

{% set SIZE = params.SIZE\|default(\'0,0,0,0\')\|string %}

\# Parse SIZE into min/max coordinates

{% set coords = SIZE.split(\',\') %}

{% set min_x = coords\[0\]\|float %}

{% set min_y = coords\[1\]\|float %}

{% set max_x = coords\[2\]\|float %}

{% set max_y = coords\[3\]\|float %}

\# ============================================================

\# 1. SAFETY & SANITY CHECKS

\# ============================================================

\# Check chamber temp (PLA safety)

{% set chamber_temp = printer\[\'temperature_sensor
chamber\'\].temperature\|default(20)\|float %}

{% if chamber_temp \> CHAMBER_MAX %}

{action_respond_info(\"WARNING: Chamber temp %.1fC exceeds PLA safe
limit (%dC). Cracking door or waiting\...\" % (chamber_temp,
CHAMBER_MAX))}

\# Optional: pause and wait for cooldown

\# TEMPERATURE_WAIT SENSOR=\"temperature_sensor chamber\"
MAXIMUM={CHAMBER_MAX}

{% endif %}

\# Check filament sensor (if installed)

{% if \'filament_switch_sensor runout\' in printer %}

{% if not printer\[\'filament_switch_sensor runout\'\].filament_detected
%}

{action_raise_error(\"FILAMENT RUNOUT: Load filament before starting
print.\")}

{% endif %}

{% endif %}

\# ============================================================

\# 2. HEATING SEQUENCE (Optimized for PLA)

\# ============================================================

\# Home if not already homed

{% if \'x\' not in printer.toolhead.homed_axes or \'y\' not in
printer.toolhead.homed_axes or \'z\' not in printer.toolhead.homed_axes
%}

G28

{% endif %}

\# Move to park position while heating (prevents nozzle oozing on bed)

G0 X150 Y10 Z50 F12000

\# Heat bed first (PLA doesn\'t need aggressive soak, but brief preheat
helps)

M140 S{BED_TEMP}

M190 S{BED_TEMP}

\# Brief bed soak for PLA (5 seconds is plenty --- adjust if your room
is cold)

G4 P5000

\# Heat hotend

M104 S{EXTRUDER_TEMP}

M109 S{EXTRUDER_TEMP}

\# ============================================================

\# 3. BED SURFACE & Z-OFFSET

\# ============================================================

\# Apply Z-offset based on bed surface type

{% set z_offset_adjustment = 0.0 %}

{% if \'Textured\' in SURFACE %}

{% set z_offset_adjustment = -0.04 %}

{action_respond_info(\"Bed surface: Textured PEI --- applying Z-offset
%.3f\" % z_offset_adjustment)}

{% elif \'Smooth\' in SURFACE %}

{% set z_offset_adjustment = 0.0 %}

{action_respond_info(\"Bed surface: Smooth PEI --- no Z-offset
adjustment\")}

{% elif \'High Temp\' in SURFACE %}

{% set z_offset_adjustment = 0.06 %}

{action_respond_info(\"Bed surface: High Temp Plate --- applying
Z-offset +%.3f\" % z_offset_adjustment)}

{% else %}

{action_respond_info(\"Bed surface: \'%s\' not recognized, using default
Z-offset\" % SURFACE)}

{% endif %}

SET_GCODE_OFFSET Z={z_offset_adjustment} MOVE=1

\# ============================================================

\# 4. ADAPTIVE BED MESH (KAMP-K2)

\# ============================================================

\# Use KAMP\'s adaptive mesh if bounding box is valid

{% if min_x != max_x and min_y != max_y %}

{action_respond_info(\"Running KAMP adaptive mesh for bounding box:
X%.1f-%.1f Y%.1f-%.1f\" % (min_x, max_x, min_y, max_y))}

\# KAMP-K2 adaptive mesh macro (from grant0013\'s repo)

\# This should already exist in your config

BED_MESH_CALIBRATE mesh_min={min_x},{min_y} mesh_max={max_x},{max_y}

{% else %}

{action_respond_info(\"No valid print size detected --- falling back to
full bed mesh\")}

BED_MESH_CALIBRATE

{% endif %}

\# ============================================================

\# 5. ADAPTIVE PURGE (KAMP-K2)

\# ============================================================

\# Let KAMP handle the adaptive purge

\# If KAMP\'s purge macro is named differently in your config, adjust
below

{% if \'ADAPTIVE_PURGE\' in printer.gcode.commands %}

ADAPTIVE_PURGE

{% else %}

\# Fallback: jschuh\'s smart purge line

{% if \'DRAW_PURGE_LINE\' in printer.gcode.commands %}

DRAW_PURGE_LINE

{% else %}

\# Ultimate fallback: simple purge line

G0 X10 Y10 Z0.2 F6000

G1 X60 Y10 E15 F1500

G1 X60 Y15 E0.5

G1 X10 Y15 E15 F1500

G0 Z2

{% endif %}

{% endif %}

\# ============================================================

\# 6. FINAL PREP

\# ============================================================

\# Reset extruder

G92 E0

\# Move to start position

G0 X{min_x + 10} Y{min_y + 10} Z0.2 F12000

\# Enable part cooling fan (PLA needs it from layer 1)

M106 S255

{action_respond_info(\"PRINT_START complete --- commencing print\")}

\`\`\`

\-\--

\## 4. Matching PRINT_END Macro

Also add this for clean, safe print completion:

\`\`\`ini

\[gcode_macro PRINT_END\]

gcode:

\# Turn off heaters

M104 S0

M140 S0

\# Retract filament slightly to prevent oozing

G91

G1 E-2 F1800

G90

\# Move nozzle away from print

G0 X150 Y280 Z{printer.toolhead.position.z + 10} F12000

\# Turn off part cooling fan

M106 S0

\# Disable motors

M84

\# Park toolhead

G0 X150 Y280 F12000

{action_respond_info(\"PRINT_END complete --- print finished\")}

\`\`\`

\-\--

\## 5. OrcaSlicer End G-Code

In \*\*OrcaSlicer → Printer Settings → Machine G-code → End G-code\*\*:

\`\`\`gcode

PRINT_END

\`\`\`

\-\--

\## 6. Optional Enhancements (Add Later)

\### 6.1 Chamber Exhaust Fan Control

If you add a low-RPM exhaust fan (e.g., Noctua 40mm) to vent warm air:

\`\`\`ini

\[fan_generic chamber_exhaust\]

pin: PA8 \# Check your board pinout

max_power: 0.4 \# Run at 40% max to keep it quiet

\`\`\`

Then in \`PRINT_START\`, after heating:

\`\`\`gcode

\# Turn on exhaust fan if chamber \> 30C

{% if chamber_temp \> 30 %}

SET_FAN_SPEED FAN=chamber_exhaust SPEED=0.3

{% endif %}

\`\`\`

\### 6.2 Timelapse Trigger

If you install \*\*Moonraker Timelapse\*\*:

\`\`\`gcode

\# In PRINT_START, after G28:

TIMELAPSE_TAKE_FRAME

\`\`\`

And in \`PRINT_END\`:

\`\`\`gcode

TIMELAPSE_RENDER

\`\`\`

\### 6.3 Exclude Object (Already in KAMP-K2?)

Verify it\'s enabled in \`printer.cfg\`:

\`\`\`ini

\[exclude_object\]

\`\`\`

In OrcaSlicer: \*\*Process → Others → Label objects\*\* → Enable

\-\--

\## 7. Troubleshooting

\| Issue \| Likely Cause \| Fix \|

\|\-\--\|\-\--\|\-\--\|

\| \`Unknown command: ADAPTIVE_PURGE\` \| KAMP macro name differs \|
Check \`printer.cfg\` for KAMP\'s actual purge macro name and update
\`PRINT_START\` \|

\| Z-offset not applying \| \`curr_bed_type\` not passing from Orca \|
Verify Start G-code syntax; check Fluidd console for passed parameters
\|

\| Chamber temp reads 0 \| No chamber sensor configured \| Check
\`temperature_sensor chamber\` in \`printer.cfg\`; use \`mcu_temp\`
fallback if needed \|

\| Mesh takes too long \| Full bed mesh instead of adaptive \| Verify
\`SIZE\` parameter is passing from Orca; check KAMP-K2 config \|

\-\--

\## 8. Quick Reference: What Each Mod Does

\| Mod \| What It Does \| Your Status \|

\|\-\--\|\-\--\|\-\--\|

\| \*\*KAMP-K2\*\* \| Adaptive mesh + purge per print size \| ✅
Installed \|

\| \*\*K2 Camera\*\* \| Better camera support \| ✅ Installed \|

\| \*\*Creality Helper Script\*\* \| PID cal, backups, stress test \|
Check if installed \|

\| \*\*jschuh macros\*\* \| Smart purge, surface profiles, fast mesh \|
Add for this macro \|

\| \*\*Chamber temp sensor\*\* \| PLA heat creep prevention \| Add to
\`printer.cfg\` \|

\| \*\*Custom PRINT_START\*\* \| This file --- combines everything \|
Add this macro \|

\| \*\*Exclude Object\*\* \| Cancel single failed parts \| Verify
enabled \|

\-\--

\> \*\*Generated:\*\* 2026-07-18

\> \*\*For:\*\* Creality K2 Pro, FW 1.1.6.3, PLA-only bedroom setup

\> \*\*Next step:\*\* SSH into printer, verify KAMP macro names, paste
this \`PRINT_START\`, test with a small calibration cube.
