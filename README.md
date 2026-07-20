# Engineering Notes for the Creality K2 Pro

*A technical notebook documenting Klipper engineering decisions, experiments, and workflows.*

> **Understanding before copying.**

---

This repository documents the engineering decisions, experiments, configurations, and workflows developed for the **Creality K2 Pro** running **Klipper**.

Rather than collecting macros or configuration files, these notes explain the reasoning behind each solution, the assumptions under which it was developed, and the trade-offs that influenced its design.

The goal is simple:

> **Help readers understand a solution before asking them to use it.**

---

## How to Read This Repository


These Engineering Notes are intended to be read sequentially.

If you are new to the project, begin with this README and continue with
TN-001, which introduces the overall design objectives and engineering
approach.

The remaining Technical Notes describe individual components of the
implementation. Although they can be consulted independently as reference
material, reading them in order provides the context behind each design
decision.

## Current Technical Notes

| ID | Title | Status |
|----|-------|--------|
| TN-001 | [PLA Bedroom PRINT_START](docs/TN-001-PLA_Bedroom_PRINT_START.md) | Draft |

---

## Philosophy

Each Technical Note aims to:

- Explain the problem.
- Describe the assumptions.
- Present the solution.
- Discuss the limitations.
- Document future improvements.

> **Understanding always comes before copying.**

---

## Scope

This repository will gradually document:

- Klipper macros
- OrcaSlicer profiles
- Print workflows
- Calibration procedures
- Hardware modifications
- Firmware observations
- Performance tuning
- Troubleshooting notes
  
---


## Repository Status

There are often several valid engineering solutions to the same problem.
These notes explain the reasoning behind one approach so that readers can
evaluate, adapt, or improve it for their own needs.

This repository is under active development.

Future revisions will refine the structure, improve the explanations, and
extract reusable components while preserving the engineering rationale behind
each design decision.

> *Engineering is a process of continuous refinement. Documentation should
> reflect that process.*
