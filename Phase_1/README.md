# Phase 1 — UAV Fundamentals & Survey / Literature Review

Pre-design research and market study. Goal: build a clear understanding of drone systems, hardware, firmware, and architectures, and produce structured comparison matrices that will inform Phase 2 hardware design.

> FPV drones are used as the initial learning platform — simplest and most modular drone systems to build and modify.

---

## Contents

| # | Folder | Topic | Status |
|--:|--------|-------|:------:|
| 00 | [Fundamentals](./00_Fundamentals) | How drones fly, UAV types, glossary, master resource list | 🟡 |
| 01 | [FPV Drones](./01_FPV_Drones) | Anatomy of an FPV build — frame, motors, props, radio, VTX, power | 🟡 |
| 02 | [Flight Controllers](./02_Flight_Controllers) | Pixhawk ecosystem, FPV FCs, FC internals, comparison matrix | 🟡 |
| 03 | [ESCs](./03_ESCs) | Architectures, 4-in-1 vs individual, firmwares, comparison matrix | 🟡 |
| 04 | [Firmwares](./04_Firmwares) | Betaflight · INAV · Ardupilot · PX4 | 🟡 |
| 05 | [AIO vs Stack](./05_AIO_vs_Stack) | When AIO FC+ESC beats a separate FC+ESC stack | 🟡 |
| 06 | Autonomous Stacks | Companion computers, MAVLink/MAVSDK, ROS integration | 🔴 |
| — | matrices/ | CSV copies of all comparison tables | 🔴 |
| — | [assets/](./assets) | Diagrams, photos, screenshots | — |

## Phase 1 deliverables

1. Comparison & evaluation matrices for FCs, ESCs, firmwares, and other components.
2. A written **report** summarising findings.
3. A **PPT** presenting the survey.

## Phase 2 targets (informed by this research)

- **System 1 — General Commercial-Grade FC:** for photography / surveillance / load-carrying drones. Reliable, redundant sensors, stable autonomous flight, industry-grade design.
- **System 2 — Lightweight AIO FC + ESC:** for lightweight surveillance and compact autonomous UAVs. Low weight, compact, good thermal handling, power-efficient.

## Status legend
🟢 done · 🟡 in progress · 🔴 not started
