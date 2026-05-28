# Firmware — Comparison Matrix

CSV mirror: [`../matrices/firmwares.csv`](../matrices/firmwares.csv)

---

## A. The four open-source firmwares

| Firmware | License | RTOS | Vehicles | Autonomy | EKF | GCS | Best for |
|----------|---------|------|----------|----------|-----|-----|----------|
| **Betaflight** | GPLv3 | None (bare-metal) | Multirotor only | ❌ (GPS Rescue only) | None (complementary filter) | Betaflight Configurator | FPV racing / freestyle / cinematic |
| **INAV** | GPLv3 | None (bare-metal) | Multirotor + fixed-wing + VTOL | ✅ basic | None (complementary) | INAV Configurator | Hobby autonomy, fixed-wing, long-range |
| **ArduPilot** | **GPLv3** | ChibiOS | Copter / Plane / Rover / Sub / Heli / Blimp / AntennaTracker | ✅ **mature** | EKF3 (24-state) | Mission Planner, QGroundControl | Industrial multirotor, mapping, ag-spray |
| **PX4** | **BSD-3 / Apache-2.0** | NuttX | Multirotor / Plane / VTOL / Rover | ✅ **mature** | EKF2 | QGroundControl | Commercial products, ROS 2 research, defense |

---

## B. Performance / capability detail

| Capability | Betaflight | INAV | ArduPilot | PX4 |
|------------|:---:|:---:|:---:|:---:|
| Bidirectional DShot + RPM filter | ✅ | ✅ | ✅ | ✅ |
| Dynamic notch filter | ✅ | ✅ | ✅ | ✅ |
| 8 kHz PID loop | ✅ | ✅ | 1–2 kHz | 1 kHz |
| Waypoint missions | ❌ | ✅ | ✅ | ✅ |
| RTH (Return to Home) | partial | ✅ | ✅ | ✅ |
| Position hold (GPS / optical flow) | ❌ | ✅ | ✅ | ✅ |
| Auto-takeoff / auto-landing | ❌ | ✅ basic | ✅ | ✅ |
| Failsafe state machine | ✅ basic | ✅ | ✅ rich | ✅ rich |
| Sensor redundancy (multi-IMU) | ❌ | ❌ | ✅ | ✅ |
| Sensor failure auto-recovery | ❌ | ❌ | ✅ via EKF lane switching | ✅ via EKF lane switching |
| Smart battery (SMBus / DroneCAN) | ❌ | ❌ | ✅ | ✅ |
| Companion-computer MAVLink | partial | ✅ | ✅ | ✅ |
| ROS 1 / ROS 2 integration | ❌ | ❌ | ✅ MAVROS | ✅ **native (uXRCE-DDS)** |
| Vision-based position (VIO, VICON) | ❌ | ❌ | ✅ | ✅ |
| Simulator (SITL) | partial | partial | ✅ excellent | ✅ excellent |
| OTA firmware update | ❌ | ❌ | ✅ (some boards) | ✅ |

---

## C. Licensing implications (the most-asked decision criterion)

| Firmware | If I ship a drone with this firmware, must I publish my changes? |
|----------|------------------------------------------------------------------|
| Betaflight | Yes (GPLv3) — but it's almost always vanilla |
| INAV | Yes (GPLv3) — same |
| **ArduPilot** | **Yes (GPLv3) — firmware mods must be open. Companion-computer code is unaffected.** |
| **PX4** | **No (BSD/Apache) — closed-source forks are legal.** |

For Phase 2 design that you might commercialise: **PX4** gives the most legal flexibility.

---

## D. Hardware compatibility

| Firmware | Pixhawk (FMU) | FPV FCs (Kakute, SpeedyBee) | Custom STM32H7 board |
|----------|:---:|:---:|:---:|
| Betaflight | ⚠️ Not officially | ✅ Default | ✅ Easy port |
| INAV | ⚠️ Not officially | ✅ Many | ✅ Easy port |
| ArduPilot | ✅ All FMUs | ⚠️ Limited | ✅ Add `hwdef.dat` |
| PX4 | ✅ All FMUs | ⚠️ Very limited | ✅ Add `boards/<vendor>/<board>/` |

For Phase 2: **System 1 ships as a Pixhawk FMUv6X-conformant board** — ArduPilot + PX4 will both compile with a board definition. **System 2 (AIO FPV)** ships as Betaflight + INAV.

---

## E. Decision tree

```
Is the drone a multirotor for FPV / acro flight?
├── Yes → Betaflight
└── No → continue
    Is it a fixed-wing / VTOL hobby project?
    ├── Yes → INAV
    └── No → continue
        Is it a commercial product where IP matters?
        ├── Yes (closed source required) → PX4
        └── No → ArduPilot (widest community, most mature for everything else)
```

---

## F. For Phase 2 — recommended dual-stack

| Phase 2 system | Primary firmware | Secondary firmware |
|----------------|------------------|---------------------|
| **System 1 (Commercial FC, FMUv6X)** | **PX4** (BSD license) | **ArduPilot** (broader community) — design supports both |
| **System 2 (Lightweight AIO)** | **Betaflight** | INAV (alternate build target on same hardware) |

This way each board has a single primary firmware identity but an easy alternative for users who prefer the other.

---

## Sources
1. Betaflight GitHub — https://github.com/betaflight/betaflight
2. INAV GitHub — https://github.com/iNavFlight/inav
3. ArduPilot GitHub — https://github.com/ArduPilot/ardupilot
4. PX4 GitHub — https://github.com/PX4/PX4-Autopilot
5. ThinkRobotics — *PX4 vs ArduPilot 2026* — https://thinkrobotics.com/blogs/learn/px4-vs-ardupilot-complete-comparison-guide-for-drone-developers
6. The Droning Company — *PX4 vs ArduPilot* — https://thedroningcompany.com/blog/px4-vs-ardupilot-choosing-the-right-open-source-flight-stack
