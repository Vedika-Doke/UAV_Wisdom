# ArduPilot

> The mature, vehicle-agnostic, "everything" open-source autopilot. Originated in 2010 as **APM** (ArduPilot Mega) on an Arduino board. Now the largest open-source flight stack by line-count and contributor base.

If your application is reliability + breadth-of-vehicles, this is the default choice. Companies running cargo, mapping, agricultural, and inspection drones at scale tend to be on ArduPilot.

---

## 1. Vehicles supported (this is its main differentiator)

| Vehicle | Description |
|---------|-------------|
| **Copter** | Multirotor (3, 4, 6, 8 motors; coaxial; H-frame; Y6; X8…) |
| **Plane** | Fixed-wing, including VTOL ("Q-Plane") |
| **Rover** | Ground robots, boats |
| **Sub** | Underwater ROVs |
| **Heli** | Traditional helicopter (single + tail rotor, including dual-rotor coax) |
| **Blimp** | Lighter-than-air |
| **AntennaTracker** | Pan/tilt tracking dish for telemetry / video |

No other open-source autopilot covers this range in one codebase.

---

## 2. Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  ArduPilot main loop (on ChibiOS RTOS, or POSIX for SITL)   │
│                                                              │
│   AP_Vehicle (Copter / Plane / Rover / Sub / ...)            │
│   ├── Mode controller (Stabilize, Loiter, Auto, RTL, etc.)   │
│   ├── EKF3 (state estimator)                                 │
│   ├── AC_AttitudeControl / AC_PosControl                     │
│   ├── Motor mixer                                            │
│   └── Output to PWM/DShot/DroneCAN                           │
│                                                              │
│   AP_HAL (Hardware Abstraction Layer)                        │
│   ├── ChibiOS (STM32 family)                                 │
│   ├── Linux (companion / Navio2 / Pi)                        │
│   └── SITL (simulator on Linux/macOS)                        │
└─────────────────────────────────────────────────────────────┘
```

The **AP_HAL** layer is the key architectural decision — it lets the same vehicle code run on a Pixhawk *or* on Linux *or* in pure simulation (SITL). Drivers are written once per HAL.

**RTOS:** ChibiOS (lightweight, deterministic) on hardware; bare Linux for SITL.

**State estimator:** EKF3 — 24-state Extended Kalman Filter fusing IMU, GPS, baro, mag, optical-flow, rangefinder, visual-inertial odometry, wheel encoders, etc. Multiple EKF instances run in parallel for IMU failure tolerance.

---

## 3. Strengths

- **Most vehicle types supported** in one codebase.
- **Most mature** — ~15 years of production deployment.
- **Best EKF** (arguably) — EKF3 handles a wide range of sensor failure modes gracefully.
- **Most parameters exposed** — every behavior is tunable (often *too* tunable for novices).
- **Two excellent GCS options:** **Mission Planner** (Windows, Microsoft .NET, the original) and **QGroundControl** (cross-platform).
- **Massive community** — `discuss.ardupilot.org` is responsive; partner manufacturers ship pre-configured solutions.
- **DroneCAN** is a first-class citizen — many smart peripherals only target ArduPilot.

---

## 4. Weaknesses

- **GPLv3 license** — modifications you ship must be open-sourced. This is a deal-breaker for some commercial products (though *companion-computer code* is unaffected). See **PX4 vs ArduPilot license** below.
- **Steeper learning curve** than INAV/Betaflight.
- **More parameters** = more rope to hang yourself with.
- Codebase is C++; large, dense, with some legacy from the Arduino era.
- ROS 2 integration exists (via MAVROS or ArduPilot's DDS) but is less native than PX4's micro-ROS support.

---

## 5. License: GPLv3 (the practical implication)

If you ship a drone product running ArduPilot and modify the firmware:
- You must **publish your firmware modifications** under GPL.
- Your **companion computer code is NOT affected** — only the firmware itself.
- Your **hardware** is unaffected — you can sell closed-source PCBs running open firmware.

In practice: **most company IP lives on the companion computer or in mission logic**, not in the autopilot itself, so GPLv3 is rarely a blocker. But for **defense / export-controlled** products that legally cannot publish source, PX4's BSD license wins.

---

## 6. Hardware support

- All Pixhawk FMU revs (v2, v3, v4, v5, v5X, v6C, v6X, v6X-RT).
- Many non-Pixhawk boards: CubePilot, Matek, MRO, Holybro Kakute (yes, even FPV-grade FCs run ArduPilot).
- **Even Raspberry Pi**: ArduPilot can run on a Pi with a Navio2 / Erle-Brain add-on, treating the Pi as both autopilot and companion.

Build system: `waf` (Python-based). `./waf configure --board=Pixhawk6X` then `./waf copter`.

---

## 7. Tooling

- **Mission Planner** (Windows) — the original GCS, .NET-based, very feature-rich.
- **QGroundControl** — cross-platform alternative.
- **MAVProxy** — terminal-based, for headless / SITL use.
- **SITL** — software-in-the-loop simulator. Run the full firmware on your laptop against a physics sim. Best test environment in the industry.
- **Log analysis** — `MAVExplorer.py`, `pymavlink`, and online tools like `https://plot.ardupilot.org`.

---

## 8. When to use ArduPilot

| If the drone is... | Use ArduPilot? |
|--------------------|:--------------:|
| Multirotor cargo / inspection | ✅ |
| Fixed-wing mapping | ✅ |
| Heavy-lift agricultural sprayer | ✅ (industry standard) |
| Surveillance / patrol with waypoints | ✅ |
| Need every conceivable failsafe path | ✅ |
| Research with ROS 2 | ⚠️ PX4 is more native |
| Defense / closed-source product | ❌ → PX4 (BSD license) |
| Pure FPV racing | ❌ → Betaflight |

---

## 9. For Phase 2

ArduPilot is the **primary firmware target** for **System 1 (Commercial FC)** because:
- Most mature autopilot for the brief's vehicle types (photography, surveillance, load-carrying).
- Best EKF for multirotor position estimation.
- Largest community → debugging support during integration.
- Best DroneCAN ecosystem → smart ESCs, smart batteries, RTK GPS.

For the design:
- Target an existing supported board profile (e.g., Pixhawk 6X) and *add* a board definition for the custom FC.
- Use `hwdef.dat` to describe the custom board to ArduPilot's build system.
- Plan to upstream the board definition to ArduPilot's `libraries/AP_HAL_ChibiOS/hwdef/` directory.

A board definition is ~200 lines of `hwdef.dat` declaring pin functions, SPI/I²C buses, motor outputs, telemetry ports. Doable in a day if the hardware follows Pixhawk conventions.

## Sources
1. ArduPilot.org — https://ardupilot.org/
2. ArduPilot GitHub — https://github.com/ArduPilot/ardupilot
3. ArduPilot Documentation — https://ardupilot.org/copter/index.html
4. ArduPilot EKF3 Docs — https://ardupilot.org/dev/docs/extended-kalman-filter.html
5. ArduPilot board definition guide — https://ardupilot.org/dev/docs/porting.html
6. PX4 vs ArduPilot 2026 — https://thinkrobotics.com/blogs/learn/px4-vs-ardupilot-complete-comparison-guide-for-drone-developers
7. ArduPilot Discourse — https://discuss.ardupilot.org/
