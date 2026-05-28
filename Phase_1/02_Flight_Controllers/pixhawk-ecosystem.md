# Pixhawk Ecosystem

The most important thing to understand about Pixhawk is that **it is not a product** — it is a **set of open hardware specifications**. Anyone can build a board that conforms to a Pixhawk FMU standard, and any firmware (PX4 or ArduPilot) compiled for that FMU revision will run on it.

This is exactly the model you'd find in PC hardware (ATX motherboards) or in 3D printers (RAMPS, Duet) — but applied to drone autopilots.

---

## 1. Origin and governance

- **Origin (2008–2010):** PX4 project at ETH Zurich (Lorenz Meier and team).
- **Hardware standard:** "Pixhawk" — the reference autopilot designed alongside the PX4 firmware.
- **Today:** Stewarded by the **Dronecode Foundation** (under the Linux Foundation umbrella). Standards are developed in public by the **Pixhawk Special Interest Group**.
- **Members / sponsors:** Auterion, NXP, Freefly Systems, Holybro, CUAV, ModalAI, Skydio (historically), and others.

The same project produced three pieces of de-facto standard infrastructure for the drone industry:
1. **PX4** — the autopilot firmware
2. **MAVLink** — the messaging protocol used between FC, GCS, and companion computers
3. **QGroundControl (QGC)** — the cross-platform GCS

So the "Pixhawk ecosystem" really means: open hardware FMUs + PX4/ArduPilot firmware + MAVLink + QGC. All four pieces work together.

---

## 2. The three standards areas

| Area | What it covers |
|------|----------------|
| **Autopilot** | Flight Management Unit (FMU) — the FC board itself. Defines connectors, power, sensor expectations. |
| **Payload** | Standardised electrical/mechanical interface between FC and cameras/gimbals/actuators. |
| **Smart Battery** | Battery Management System (BMS) interfaces — cell voltages, state-of-charge, cycle count over SMBus / DroneCAN. |

For Phase 2, the relevant standard is **Autopilot** — specifically picking an FMU revision to target.

---

## 3. FMU versions (timeline)

| Rev | Year | MCU | Clock | Flash / RAM | Sensor approach | Used in |
|-----|------|-----|------:|-------------|-----------------|---------|
| FMUv1 | 2011 | STM32F405 | 168 MHz | 1 MB / 192 KB | Single IMU | Original PX4 board |
| FMUv2 | 2014 | STM32F427 | 180 MHz | 2 MB / 256 KB | Single IMU | Pixhawk 1, Pixhawk Mini |
| FMUv3 | 2015 | STM32F427 rev 3 | 180 MHz | 2 MB / 256 KB | Dual IMU | The Cube (early) |
| FMUv4 | 2016 | STM32F427 | 180 MHz | 2 MB / 256 KB | Improved sensors | Pixracer |
| FMUv5 | 2018 | STM32F765 | 216 MHz | 2 MB / 512 KB | Triple IMU | Pixhawk 4 |
| **FMUv5X** | 2020 | STM32F765 | 400 MHz | 2 MB / 512 KB | Triple IMU, isolated buses | Pixhawk 5X |
| **FMUv6C** | 2023 | STM32H743 | 480 MHz | 2 MB / 1 MB | Dual IMU (cost-down) | Pixhawk 6C |
| **FMUv6X** | 2023 | STM32H753 | 480 MHz | 2 MB / 1 MB | **Triple IMU + isolated power domains** | Pixhawk 6X, CUAV V6X |
| **FMUv6X-RT** | 2024 | **NXP iMX RT1176** | **1 GHz** | 64 MB / 2 MB | Triple IMU | Holybro Pixhawk 6X-RT |

### Big architectural inflection: FMUv5X
v5X introduced the **modular** approach Pixhawk still uses today: the autopilot is split into separate **IMU**, **FMU**, and **Base** modules connected by standardised **100-pin** and **50-pin** "Pixhawk Bus" connectors. This means:
- IMU can be vibration-isolated mechanically (it's a daughterboard).
- The same FMU module can be paired with different base boards (mini, standard, custom carrier).
- Vendors can innovate on connectors / IO without changing the FMU.

### Big jump: FMUv6X-RT
NXP **iMX RT1176** dual-core (Cortex-M7 @ 1 GHz + Cortex-M4 @ 400 MHz) replaces the STM32. This is closer to a small applications processor than a classic MCU — gives serious headroom for advanced estimators, EKF, computer vision pre-processing on the autopilot itself.

---

## 4. Boards from major vendors

All of these conform to a Pixhawk FMU standard, so PX4 and ArduPilot will run on them.

| Vendor | Board | FMU rev | Sensors | Notable |
|--------|-------|---------|---------|---------|
| **Holybro** | Pixhawk 6C | FMUv6C | Dual IMU (ICM-42688-P + BMI055), baro | Cost-effective, $200 class |
| Holybro | Pixhawk 6X | FMUv6X | **Triple IMU** (ICM-45686 + ICM-42688 + BMI088), 2× baro | Industrial workhorse |
| Holybro | Pixhawk 6X-RT | FMUv6X-RT | Same triple-IMU | iMX RT1176, future-proof |
| **CUAV** | Pixhawk V6X | FMUv6X | **Triple IMU** + 2 baros, separated domains | Direct competitor to Holybro 6X, ethernet |
| **CubePilot** | The Cube Orange+ | (Cube standard, FMUv3-compat carrier) | **Triple IMU** (2 vibe-isolated + 1 reference) | Bullet-proof, used widely in industry |
| **mRo** | Pixracer Pro / Control Zero | FMUv5/v6 derivatives | Dual IMU | Niche/military variants |
| **ModalAI** | VOXL2 | (not pure FMU, integrated companion) | — | All-in-one with compute |

> **Note for Phase 2:** for the **Commercial Grade FC** (System 1), targeting **FMUv6X** is the highest-leverage choice — it's the current industrial standard, has the most pre-built BSP and PX4/ArduPilot support, and offers the redundancy the brief explicitly calls for.

---

## 5. What "FMU-conformant" actually requires

If you want to build a custom board that PX4 will recognise as e.g. FMUv6X:

- **MCU** must be STM32H753 (or compatible H7 variant in the standard).
- **Sensor set** must match: triple IMU (ICM-42688P + ICM-45686 + BMI088 typical), dual baro (BMP388 + DPS310), magnetometer (typically external on GPS), FRAM for parameter storage.
- **Power architecture** with isolation domains: separate regulators for FMU vs IMU vs IO.
- **Connectors** match the Pixhawk Bus pinout (100-pin + 50-pin for v6X modular form factor) — or the older monolithic connector set for v5/older.
- **Bootloader** with the standard upload protocol (px_uploader.py).
- Optionally pass conformance testing through the Dronecode Foundation.

Then PX4 builds with `make px4_fmu-v6x_default` will produce a binary that flashes and runs.

---

## 6. Reading the spec yourself

The current standards live at **https://github.com/pixhawk/Pixhawk-Standards**. PDFs for v5X, v6C, v6X are downloadable from there. **Read FMUv6X.pdf first** — it's the most relevant to Phase 2.

Each standard PDF defines:
- Mandatory peripherals & buses
- Connector pinouts (down to pin numbers)
- Voltage rails and isolation requirements
- Mechanical form factor (mm-precise)
- Status LED behaviour, safety switch, buzzer

---

## 7. Strategic takeaways for Phase 2

| Decision | Recommendation |
|----------|----------------|
| Target standard for System 1 (Commercial FC) | **FMUv6X** — modular, industrial-grade, well-supported |
| Firmware | **PX4** (Apache-2.0 — friendly for productisation) and **ArduPilot** (broader vehicle support) — design to support both via the standard |
| IMU strategy | Triple-redundant with **different vendors** (e.g., InvenSense + Bosch) so a vendor-specific bug doesn't take down all three |
| Comms | Include **DroneCAN/CAN-FD** (for smart peripherals) + **Ethernet** (for companion computer) — both are FMUv6X expectations |
| Power | Dual redundant power inputs with diode-OR; isolated rails for IMU vs IO |

For **System 2 (Lightweight AIO)** the FMU standards don't really apply — that's hobby/FPV territory, runs Betaflight/INAV, and has its own (looser) conventions covered in [fpv-fcs.md](./fpv-fcs.md).

## Sources
1. Pixhawk Standards GitHub — https://github.com/pixhawk/Pixhawk-Standards
2. Dronecode — *Latest Pixhawk Open Standards FMUv6X & FMUv6C* — https://dronecode.org/pixhawk-fmuv6-family-of-open-standards-are-now-available/
3. Dronecode — *Announcing FMUv5X* — https://dronecode.org/announcing-the-pixhawk-fmuv5x-open-standard/
4. Dronecode — *Announcing FMUv6X-RT* — https://dronecode.org/announcing-the-pixhawk-fmuv6x-rt/
5. PX4 Docs — *Pixhawk Series* — https://docs.px4.io/main/en/flight_controller/pixhawk_series
6. PX4 Docs — *Holybro Pixhawk 6X* — https://docs.px4.io/main/en/flight_controller/pixhawk6x.html
7. PX4 Docs — *CUAV V6X* — https://docs.px4.io/main/en/flight_controller/cuav_pixhawk_v6x.html
8. CubePilot Docs — *Cube specifications* — https://docs.cubepilot.org/user-guides/autopilot/the-cube/introduction/specifications
9. Holybro Pixhawk 6X — https://holybro.com/products/pixhawk-6x
10. Holybro Pixhawk 6C — https://holybro.com/products/pixhawk-6c
