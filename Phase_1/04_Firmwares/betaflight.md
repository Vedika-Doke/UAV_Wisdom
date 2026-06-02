# Betaflight

> Fork of Cleanflight (2015), itself a fork of Baseflight (2014). Focused **single-mindedly** on FPV racing and freestyle for multirotors.

If a multirotor cares about *flight feel and latency*, it runs Betaflight. If it cares about autonomy, it doesn't.

---

## 1. Philosophy

- **Acro-first.** Everything optimised for the pilot-in-the-loop case.
- **Bare-metal main loop** — no RTOS, no scheduler, no abstractions. The PID task is the heartbeat; everything else fits around it.
- **Latency over features** — they will refuse to add a feature if it costs gyro loop microseconds.
- **MultirotorOnly.** No fixed-wing, no rover, no autonomy beyond GPS Rescue.

---

## 2. Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Betaflight main loop  (bare-metal, no RTOS)                │
│                                                              │
│   ┌──Gyro task (4-32 kHz)──┐                                 │
│   │   Read IMU via DMA      │                                │
│   │   ↓                     │                                │
│   │   Filtering (notch,     │                                │
│   │     dynamic, RPM)       │                                │
│   └─────────┬───────────────┘                                │
│             ↓                                                │
│   ┌──PID task (8 kHz)──────┐                                 │
│   │   Compute error & PID   │                                │
│   │   ↓                     │                                │
│   │   Motor mixer           │                                │
│   │   ↓                     │                                │
│   │   DShot output          │                                │
│   └─────────────────────────┘                                │
│                                                              │
│   Background: RC parsing, OSD, blackbox, telemetry           │
└─────────────────────────────────────────────────────────────┘
```

Single-threaded. The "scheduler" is just a priority-ordered list of tasks; gyro and PID always come first. UART parsing, OSD updates, and blackbox writes happen in remaining cycles.

This is a feature, not a limitation. It means latency from gyro sample to motor output is deterministic — typically **<300 µs** end-to-end on an H7.

---

## 3. Strengths

| Feature | Why it matters |
|---------|----------------|
| **RPM filtering** | Notch-filters out each motor's vibration based on real-time RPM from bidirectional DShot. Game-changer for prop wash / motor heat. |
| **Dynamic notch filter** | Identifies and removes the highest vibration peak in flight, adapts continuously. |
| **D-term lowpass + dynamic LPF** | Aggressive but well-tuned filters tame noise without sacrificing response. |
| **CLI + Configurator GUI** | Massive ecosystem; every parameter exposed and documented. |
| **Blackbox** | High-rate logging to onboard SPI flash (or SD); analyze in Blackbox Explorer. |
| **GPS Rescue** | The one "autonomy" feature — returns to home on lost RC link or low battery. |
| **Profile system** | 3 PID profiles + 3 rate profiles switchable mid-flight via switch. |

---

## 4. Limitations

- **No waypoint missions** — can't fly autonomously to GPS coordinates.
- **No fixed-wing / VTOL / rover** — multirotor only.
- **No EKF** — uses a complementary filter only. Fine for hover but no proper position estimate.
- **No vision integration** — opt-out of computer-vision drones.

For autonomy, INAV is the upgrade path (same configurator family); for full autonomy, ArduPilot/PX4.

---

## 5. Hardware support

- **MCUs:** STM32 F405, F411, F722, F745, H743, H750, plus AT32F435 ("BetaFlight 4.5+").
- **IMUs:** ICM-42688-P (recommended), ICM-20689, MPU6000 (deprecated), BMI270 (works but Betaflight has advised against).
- Hundreds of targets — almost every commercial FPV FC supports Betaflight on day one.

Source code: https://github.com/betaflight/betaflight (very active; 4.5 released late 2024, 4.6 expected mid-2026).

---

## 6. Tuning workflow (one paragraph)

Most users **don't tune from scratch** — they pick a known-good "PID slot" from a tuning database (Bardwell's, UAVTech's), then make small adjustments based on Blackbox traces. Tuning steps:
1. Verify motors are balanced and not overheating.
2. Enable bidirectional DShot + RPM filter.
3. Adjust D-term LPF cutoff to remove high-frequency noise without amplifying low-freq.
4. Fine-tune P/D gains for crisp response without overshoot.
5. Lock in I gain to handle wind / imbalance.

This is **PID tuning as art-and-data** — Blackbox plots show oscillation; you change one gain at a time.

---

## 7. Configurator

The **Betaflight Configurator** is a cross-platform Electron app (Chrome OS / Win / macOS / Linux) that:
- Connects via USB to the FC.
- Lets you set every parameter (CLI command or GUI).
- Shows a live gyro graph for vibration analysis.
- Lets you flash firmware.
- Maps RC channels, sets failsafes, configures modes.

**MSP** (MultiWii Serial Protocol) is the wire protocol between Configurator and FC. Some configurators also speak MSP DisplayPort to drive digital VTX OSDs.

---

## 8. When to use Betaflight (decision rule)

| If the drone is... | Use Betaflight? |
|--------------------|:---------------:|
| FPV racing / freestyle | ✅ Default |
| Cinewhoop with manual flight | ✅ |
| Long-range FPV cruise (no waypoints) | ✅ |
| Wants to follow a GPS mission | ❌ → INAV or ArduPilot |
| Camera drone (DJI-style hover) | ❌ → ArduPilot Copter |
| Fixed-wing / VTOL | ❌ → INAV or ArduPilot Plane |
| Indoor autonomy / vision | ❌ → PX4 + ROS 2 |

---

## 9. For Phase 2

The **System 2 (Lightweight AIO)** target ships Betaflight. Implications for the FC design:

- Pick MCU on Betaflight's supported list (H743 or AT32F435 are the future-safe picks)
- IMU = ICM-42688-P (the current Betaflight gold standard)
- Provide 8+ timer channels for motor outputs
- Reserve UARTs for RX (CRSF), VTX (MSP DisplayPort), GPS (if 7"+), ESC telemetry
- Add a **define for the target** to Betaflight's source tree (upstream PR for OSS credibility)

System 1 (Commercial FC) does *not* run Betaflight — it's PX4 or ArduPilot.

## Sources
1. Betaflight GitHub — https://github.com/betaflight/betaflight
2. Betaflight Manual / Wiki — https://betaflight.com/docs/
3. Betaflight Manufacturer Design Guidelines — https://betaflight.com/docs/development/manufacturer/manufacturer-design-guidelines
4. Joshua Bardwell — *Betaflight 4.x deep-dive videos* — YouTube
5. Oscar Liang — *Betaflight PID Tuning* — https://oscarliang.com/pid/
