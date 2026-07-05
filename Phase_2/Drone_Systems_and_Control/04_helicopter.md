# Helicopter — Key Concepts Reference

> Focus: single-rotor helicopter drones with swash-plate control, aimed at AIO FC+ESC PCB design.

---

## 1. Anatomy

![Helicopter controls overview](https://commons.wikimedia.org/wiki/Special:FilePath/Helicopter_controls_diagram.svg)
*Source: Wikimedia Commons*

```
         Main Rotor Blades       ← lift + attitude control
               │ pitch links
         Swash Plate             ← servo commands → spinning blades
               │ rotor shaft
         Gearbox / Belt Drive    ← steps down motor RPM
               │
         Main Brushless Motor
         
  Fuselage → Tail Boom → Tail Rotor   ← anti-torque + yaw
```

| Component | Role |
|-----------|------|
| Swash plate (fixed + rotating) | Transfers servo commands to spinning blades |
| Pitch links | Connect rotating swash plate to each blade grip |
| Tail rotor | Cancels main rotor torque; yaw control |
| Governor / ESC | Holds constant headspeed |

---

## 2. Lift — Pitch Not RPM

Unlike multirotors (which change motor speed), helicopters keep RPM **constant** and change blade pitch instead.

- RPM change takes 100–500 ms (motor spin-up)
- Blade pitch change is nearly **instantaneous**
- The **governor** (inside the ESC) holds RPM fixed by adjusting throttle as collective load changes

---

## 3. Rotor Head Types

![Fully articulated rotor head](https://commons.wikimedia.org/wiki/Special:FilePath/Fully_articulated_main_rotor_head.svg)
*Source: Wikimedia Commons*

![Rotor joint types](https://commons.wikimedia.org/wiki/Special:FilePath/Helicopter_rotor_joints.svg)
*Source: Wikimedia Commons*

| Type | Hinges | Used in |
|------|--------|---------|
| **Fully articulated** | Flap + lead-lag + feather | Large manned helicopters |
| **Semi-rigid / Teetering** | One shared teetering hinge | Small RC/drone helis |
| **Rigid (hingeless)** | None — blade root flexes | Modern drone helis |
| **Bearingless** | None — composite flexbeam | High-performance |

> Small drone helis use **semi-rigid or rigid** — lighter, faster response. Downside: vibration goes straight to the airframe → soft-mount the IMU.

---

## 4. Control Inputs

| Input | What moves | Effect |
|-------|-----------|--------|
| **Collective** | Swash slides up/down flat | All blade pitch → altitude |
| **Longitudinal Cyclic** | Swash tilts fore/aft | Forward/back |
| **Lateral Cyclic** | Swash tilts left/right | Left/right |
| **Pedal (Yaw)** | Tail rotor pitch | Yaw |

Minimum **4 servo channels** on AIO FC: 3 swash (CCPM) + 1 tail.

---

## 5. CCPM Mixing

All 3 swash servos move together for every input. FC does the math in firmware.

![Swash plate animation](https://commons.wikimedia.org/wiki/Special:FilePath/HelicopterSwashPlate_Tilted.gif)
*Source: Wikimedia Commons*

```
       FRONT
         ↑
    S1 (0°) ●
       /   \
S3●─────────●S2
  240°     120°

Servo 1 (  0°) = Collective + 0      + Roll
Servo 2 (120°) = Collective + 0.866P − 0.5R
Servo 3 (240°) = Collective − 0.866P − 0.5R
```

All 3 PWM outputs must be on the **same hardware timer** — even 10 µs skew causes swash wobble.

---

## 6. Torque & Anti-Torque

Motor spins rotor → fuselage wants to spin the other way (Newton's 3rd law).

- Tail rotor thrust × tail boom length = cancels main rotor torque
- More collective → more torque → FC automatically adds tail pitch (**collective–yaw feedforward**)

---

## 7. Gyroscopic Precession

![Gyroscopic precession](https://commons.wikimedia.org/wiki/Special:FilePath/Gyro_Precession.PNG)
*Source: Wikimedia Commons / FAA*

A spinning rotor responds to an applied force **90° later** in the direction of rotation.

- Want disc to tilt forward → apply max blade pitch at the **right side** (CCW rotor from above)
- The swash plate has a built-in 90° phase offset to handle this
- On flybarless FC helis: configured via `H_PHANG` in ArduPilot — wrong value → controls feel diagonal

---

## 8. Headspeed & Pitch Curve

| Parameter | Typical range |
|-----------|--------------|
| Headspeed | 1500–3000 RPM |
| Collective pitch range | −5° to +12° |
| Cyclic pitch range | ±8° |
| Gear ratio (motor → rotor) | 8:1 to 12:1 |
| Tail rotor ratio | ~4–5× main rotor |

**Pitch curve** maps collective stick position to blade pitch angle:
```
0% stick  → −2°   (idle)
50% stick →  +5°  (hover)
100% stick → +12° (max climb)
```

---

## 9. Blade Flapping

![Dissymmetry of lift](https://commons.wikimedia.org/wiki/Special:FilePath/FAA_heli-manual_Dissymmetry_of_lift.PNG)
*Source: Wikimedia Commons / FAA Helicopter Flying Handbook*

In forward flight, the advancing blade sees higher airspeed → more lift → flaps up. Retreating blade → less lift → flaps down. This is **dissymmetry of lift**.

Flapping equalises the lift automatically — the disc tilts opposite to the advance direction, correcting the roll.

---

## 10. Aerodynamic Effects

### Ground Effect

![Ground effect](https://commons.wikimedia.org/wiki/Special:FilePath/Ground_effect_heli.png)
*Source: Wikimedia Commons*

Within ~1 rotor diameter of the ground: downwash is blocked → 10–20% less power for same thrust (IGE). Design the ESC for **OGE** (out of ground effect) — the harder case.

### Vortex Ring State

![Vortex ring state](https://commons.wikimedia.org/wiki/Special:FilePath/Vortex_ring_helicopter.jpg)
*Source: Wikimedia Commons*

Descending vertically at ~300–500 fpm with power applied → rotor ingests its own downwash → lift collapses suddenly.

- Adding collective **makes it worse**
- Recovery: **forward cyclic** — fly out horizontally
- FC firmware limits descent rate to prevent entry

### Translational Lift

At ~15–25 km/h forward speed, rotor exits its own downwash → 10–15% efficiency gain → helicopter pitches up slightly and yaw kicks. FC compensates automatically.

### Autorotation

![Autorotation airflow](https://commons.wikimedia.org/wiki/Special:FilePath/Airflow_in_Autorotation_HEB.jpg)
*Source: Wikimedia Commons*

Engine off → lower collective → air flows **up** through descending disc → rotor spins like a windmill → controlled glide.

Near ground: aft cyclic to flare → pull collective to cushion touchdown using stored rotor kinetic energy.

---

## 11. Stability

Helicopters are **inherently unstable** — FC must close the loop at all times (≥400 Hz).

| Issue | Cause | FC response |
|-------|-------|-------------|
| Inherently unstable | No passive restoring force | Continuous 400 Hz loop |
| Cross-coupling | Collective → torque → yaw | Feedforward terms |
| Phase lag | Servo + mechanical delay ~20–40 ms | Limits max PID gain |
| Collective–yaw | More collective → yaw disturbance | Tail pitch feedforward |

---

## 12. Vibration

| Source | Frequency | At 2000 RPM |
|--------|-----------|-------------|
| 1P (imbalance) | RPM/60 | 33 Hz |
| 2P (blade pass) | 2×RPM/60 | 67 Hz |
| Tail rotor | ~4.5×RPM/60 | ~150 Hz |

**Fix:** Soft-mount IMU + ArduPilot harmonic notch filter (`INS_HNTCH_MODE = 3`, tracks RPM dynamically).

---

## 13. Servo & ESC Requirements

| Parameter | Servo | ESC |
|-----------|-------|-----|
| Channels | 4 min (3 CCPM + 1 tail) | 1 main motor |
| Signal | 400 Hz PWM, hardware timer | — |
| Speed | < 0.08 s/60° | — |
| Voltage | 6–8.4 V (HV) | — |
| Governor | — | Required |
| BEC | — | ≥10 A (for 4 servos) |
| RPM sense | — | Required (tach or back-EMF) |

---

## 14. AIO PCB Design Checklist

### Power
- [ ] Main ESC FET bridge — sized for OGE hover + 30% margin
- [ ] Dedicated switching BEC: **6–8.4 V, ≥10 A continuous, ≥15 A peak**
- [ ] Servo rail isolated from FC logic supply
- [ ] Current sensing on ESC output (governor load estimation)

### RPM & Governor
- [ ] RPM input: 3.3 V tolerant, RC low-pass filter before MCU pin
- [ ] Governor mode: ArduPilot `H_RSC_MODE = 3`

### Attitude Sensing
- [ ] Dual IMU (e.g. ICM-42688-P + ICM-20689), soft-mounted at board CoM
- [ ] Magnetometer ≥20 mm from power traces and FETs
- [ ] Barometer for altitude + descent rate (VRS prevention)

### Servo Output
- [ ] ≥4 hardware PWM channels, **same timer peripheral** (synchronised)
- [ ] 16-bit timer for <1 µs jitter

### MCU & Communication
- [ ] STM32H7 or F7 — hardware FPU mandatory
- [ ] MAVLink UART + telemetry UART + GPS UART

### Safety
- [ ] Hardware watchdog (IWDG) — safe servo position on MCU hang
- [ ] Collective limit in firmware (blade structural limit)
- [ ] Descent rate limit — VRS prevention

---

## Sources
- [01_swash_plate.md](./01_swash_plate.md)
- [02_rotation_orientation_control.md](./02_rotation_orientation_control.md)
- [03_drone_sensors.md](./03_drone_sensors.md)
- NPTEL: Drone Systems and Control, IISc Bangalore
- FAA Helicopter Flying Handbook (images via Wikimedia Commons)
