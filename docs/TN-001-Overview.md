# TN-001 — Overview

> **Printer:** Creality K2 Pro
>
> **Firmware:** 1.1.6.3
>
> **Slicer:** OrcaSlicer
>
> **Interface:** Fluidd
>
> **Mods:** KAMP-K2 (grant0013), K2 Camera (DnG-Crafts)

## Purpose

Extract from a limited Klipper machine all we can "squeeze" to have a kind klipper, small but powerfull. Limited by the motherboard RAM. 

...

## Audience

For everyone from beginner to well develloped printer. I need more contribuitions to become a reference repository of small inputs to shape a wide but limited system

...

## Workflow at a Glance

...
>
> **Environment:** Bedroom or ehatever may be your limitation room, PLA-only, no fume-sensitive
>
> **Status:** First Release, may contain some inconsistencies
>
> **Series:** TN-001a through TN-001h

---

## Purpose

This is the front door to the TN-001 series. TN-001 is the whole
project of setting up a custom `PRINT_START` routine for a K2 Pro that
lives in a bedroom and only prints PLA. That means chamber temperature
matters, fumes matter, and you want something that just works every
time without babysitting it.

The series is split into eight short notes (TN-001a through TN-001h)
instead of one long document, so you can read the piece you need
without wading through everything else. This page tells you what's in
each one, in plain language, so you can decide where to start.

---

## Audience

You already have KAMP-K2 and K2 Camera installed on your K2 Pro, and
you're comfortable SSHing in and editing `printer.cfg`. You don't need
to be a Klipper expert — the notes explain the "why," not just the
"paste this here."

---

## Workflow at a Glance

If you're setting this up for the first time, read in order:

| Note | Covers | Read it when... |
|---|---|---|
| TN-001a | What the macro does, at a high level | You want the big picture before touching config |
| TN-001b | What to install/configure first | You're ready to prep the printer |
| TN-001c | The actual `PRINT_START` macro | You're ready to paste the macro in |
| TN-001d | The matching `PRINT_END` macro | Right after TN-001c |
| TN-001e | One line of OrcaSlicer end G-code | Wiring up the slicer |
| TN-001f | Nice-to-haves for later | Once the core setup is working |
| TN-001g | What to do when something breaks | Something breaks |
| TN-001h | A one-page cheat sheet | You just want a quick reminder |

If you already know what you need, jump straight to that letter — each
section below gives you enough context to go in cold.

---

## What's in Each Note

### TN-001a — PRINT_START Overview

The "why" note. It explains what the custom macro adds on top of your
existing KAMP-K2 setup — chamber safety, bed-surface Z-offset, faster
meshing, a smarter purge — without touching any actual config. Start
here if you want to understand the shape of the thing before you build
it.

### TN-001b — Prerequisites

Three small jobs to do before the main macro will work: pull in one
file from jschuh's macro collection (just for a purge-line fallback),
add a chamber temperature sensor, and update your OrcaSlicer start
G-code so it hands the macro the values it needs. Skipping this note
means TN-001c won't have what it needs to run.

### TN-001c — PRINT_START Macro

The main event — the full `PRINT_START` macro you paste into
`printer.cfg`. It checks the chamber is safe for PLA, heats up, figures
out your bed surface, runs an adaptive mesh sized to your actual print,
and purges before starting. This is the note you'll come back to most.

### TN-001d — PRINT_END Macro

The short partner to TN-001c: cools down, retracts a touch, parks the
toolhead, and shuts the motors off cleanly at the end of a print.

### TN-001e — OrcaSlicer End G-Code

One line. Literally just tells OrcaSlicer to call `PRINT_END` when a
print finishes. Quick to apply once TN-001d is in place.

### TN-001f — Optional Enhancements

Extras you can add once the core setup is solid: an exhaust fan for
the chamber, a timelapse trigger, and a reminder to double-check
exclude-object support. Nothing here is required — treat it as a
"maybe later" list.

### TN-001g — Troubleshooting

A short table matching common symptoms (an unknown command, a Z-offset
that won't apply, a mesh that takes forever) to their likely cause and
fix. Check here first if something doesn't behave the way TN-001c
describes.

### TN-001h — Quick Reference

A one-page status table of every mod involved in this setup and
whether you've installed it yet, plus a short "what's next" reminder.
Good for a quick glance without re-reading everything.

---

## See Also

- [TN-001a — PRINT_START Overview](TN-001a-Macro_Overview.md)
- [TN-001b — Prerequisites](TN-001b-Prerequisites.md)
- [TN-001c — PRINT_START Macro](TN-001c-PRINT_START_Macro.md)
- [TN-001d — PRINT_END Macro](TN-001d-PRINT_END_Macro.md)
- [TN-001e — OrcaSlicer End G-Code](TN-001e-OrcaSlicer_End_Gcode.md)
- [TN-001f — Optional Enhancements](TN-001f-Optional_Enhancements.md)
- [TN-001g — Troubleshooting](TN-001g-Troubleshooting.md)
- [TN-001h — Quick Reference](TN-001h-Quick_Reference.md)
