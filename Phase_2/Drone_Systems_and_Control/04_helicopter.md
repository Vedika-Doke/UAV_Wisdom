# Helicopter — Key Concepts Reference

> Focus: single-rotor helicopter drones with swash-plate control, aimed at AIO FC+ESC PCB design.

---

## 1. Anatomy

![Helicopter controls overview](https://commons.wikimedia.org/wiki/Special:FilePath/Helicopter_controls_diagram.svg)
*Source: Wikimedia Commons — Helicopter controls diagram*

```
         ┌──────────────────────────────────────┐
         │         Main Rotor Blades            │  ← generate lift + control thrust vector
         └──────────────────┬───────────────────┘
                            │ pitch links
         ┌──────────────────┴───────────────────┐
         │   Rotating Swash Plate               │  ← spins with rotor shaft
         │   Fixed Swash Plate                  │  ← connected to servos (static)
         └──────────────────┬───────────────────┘
                            │ rotor shaft
         ┌──────────────────┴───────────────────┐
         │         Gearbox / Belt Drive         │  ← steps down motor RPM → rotor RPM
         └──────────────────┬───────────────────┘
         ┌──────────────────┴───────────────────┐
         │        Main Brushless Motor          │  ← single high-current drive
         └──────────────────────────────────────┘
  Fuselage → Tail Boom ──────────────────→ Tail Rotor  ← anti-torque + yaw
```

| Component | Role |
|-----------|------|
| Main rotor blades | Generate lift; pitch varies via swash plate |
| Swash plate (fixed + rotating) | Transfers servo commands to spinning blades |
| Pitch links | Push-rod from rotating swash plate to each blade grip |
| Collective servo | Slides swash plate up/down → all blades same pitch |
| Cyclic servos ×2 | Tilt swash plate → sinusoidal pitch variation per revolution |
| Tail rotor | Cancels main rotor reaction torque; yaw control |
| Governor / ESC | Holds constant headspeed regardless of collective load |

---

## 2. How Lift Is Generated

**The lift equation:**
```
L = ½ · ρ · v² · A · C_L(α)
  ρ = air density   v = blade tip speed   A = disc area   C_L = lift coefficient (function of pitch angle α)
```

**Key insight — helicopters control lift via pitch, not RPM:**
- Multirotor: RPM changes → lift changes (slow, 100–500 ms)
- Helicopter: RPM is **constant** (governed); blade pitch α changes → C_L changes → lift changes (nearly instantaneous)

Blade tip speed at 2000 RPM with 0.5 m radius ≈ **105 m/s (Mach 0.3)** — small tracking errors cause large vibration at this speed.

---

## 3. Rotor Head Types

![Fully articulated rotor head diagram](https://commons.wikimedia.org/wiki/Special:FilePath/Fully_articulated_main_rotor_head.svg)
*Source: Wikimedia Commons — Fully articulated main rotor head*

![Rotor head hinge types](https://commons.wikimedia.org/wiki/Special:FilePath/Helicopter_rotor_joints.svg)
*Source: Wikimedia Commons — Helicopter rotor joints*

```
FULLY ARTICULATED            SEMI-RIGID / TEETERING        RIGID (HINGELESS)

  blade ─ flap hinge         blade ─ teeter hinge          blade ─ flex root
           │                          │                              │
  blade ─ lead-lag hinge     blade ──┘  (both flap         blade ──┘  (no hinge)
           │                           together)
  feather bearing
  3 hinges per blade         1 shared hinge, 2-blade        0 hinges — stiff
  Max flexibility            Simple, lightweight            Fastest servo response
```

| Type | Hinges | Response | Used in |
|------|--------|----------|---------|
| **Fully articulated** | Flap + lead-lag + feather | Smooth | Large manned helicopters |
| **Semi-rigid / Teetering** | One teetering (2-blade) | Medium | Small RC/drone helis |
| **Rigid (hingeless)** | None — blade root flex | Fast | Modern drone helis |
| **Bearingless** | None — composite flexbeam | Fastest | High-performance |

> Small autonomous helis use **semi-rigid or rigid** — simpler, lighter, faster attitude loop. Downside: vibration goes straight to the airframe → must soft-mount IMU.

**Feathering** = blade pitch change (useful — swash plate controlled).  
**Flapping** = blade up/down motion (aerodynamic response to unequal lift).  
**Lead-lag** = blade fore/aft motion (Coriolis, creates N×RPM vibration).

---

## 4. Control Inputs

| Input | Mechanism | Primary effect | Secondary |
|-------|-----------|----------------|-----------|
| **Collective** | Swash slides straight up/down | All blade pitch → altitude | More torque → yaw |
| **Longitudinal Cyclic** | Swash tilts fore/aft | Disc tilts → forward/back | Slight climb/descent |
| **Lateral Cyclic** | Swash tilts left/right | Disc tilts → left/right | Slight climb/descent |
| **Pedal (Yaw)** | Tail rotor blade pitch | Anti-torque force → yaw | Slight roll |

**How cyclic works:**
```
Swash plate tilted forward:
  Blade at rear (180°) → high pitch → more lift → flaps up (90° later = left)
  Blade at front (0°)  → low pitch → less lift  → flaps down (90° later = right)
  Net: disc tilts forward → helicopter pitches forward → moves forward
```
> The 90° delay is **gyroscopic precession** — see §7.

**Min 4 servo channels on AIO FC:** 3 swash (CCPM) + 1 tail.

---

## 5. CCPM — Cyclic/Collective Pitch Mixing

All 3 swash servos move simultaneously for every input. FC firmware does the math every cycle.

![Swash plate tilting animation](https://commons.wikimedia.org/wiki/Special:FilePath/HelicopterSwashPlate_Tilted.gif)
*Source: Wikimedia Commons — Helicopter swash plate tilted*

```
       FRONT
         ↑
    S1 (0°) ●
       /   \
      /     \
S3●─────────●S2
  240°     120°
```

**120° CCPM equations:**
```
Servo 1 (  0°): Collective + Pitch×sin(  0°) + Roll×cos(  0°)  =  Collective + 0     + Roll
Servo 2 (120°): Collective + Pitch×sin(120°) + Roll×cos(120°)  =  Collective + 0.866P − 0.5R
Servo 3 (240°): Collective + Pitch×sin(240°) + Roll×cos(240°)  =  Collective − 0.866P − 0.5R
```

**Worked example** — Pitch command +200 µs, Roll = 0, Collective = 1500 µs (centre):
```
Servo 1: 1500 + 0     = 1500 µs  (perpendicular to pitch — no change)
Servo 2: 1500 + 173   = 1673 µs  (moves up)
Servo 3: 1500 − 173   = 1327 µs  (moves down)
→ Swash plate tilts in pitch direction ✓
```

> **AIO PCB:** All 3 PWM outputs must come from the **same hardware timer** — even 10 µs skew causes visible swash wobble.

---

## 6. Torque & Anti-Torque

Newton's 3rd law: motor spins rotor → fuselage reacts with opposite spin.

```
Anti-torque moment = F_tail × L_tail_boom

More collective → more main rotor torque → FC must add tail pitch automatically
→ collective-to-yaw feedforward in the FC PID
```

| Tail type | Yaw response | Notes |
|-----------|-------------|-------|
| Variable pitch (servo) | ~30 ms | Standard on drone helis |
| Fixed pitch (variable RPM) | ~200 ms | Simpler but slow |
| Fenestron (shrouded fan) | ~30 ms | Quieter, safer |

---

## 7. Gyroscopic Precession

![Gyroscopic precession](https://commons.wikimedia.org/wiki/Special:FilePath/Gyro_Precession.PNG)
*Source: Wikimedia Commons / FAA — Gyroscopic precession*

A spinning rotor responds to an applied force **90° later in the direction of rotation**.

```
Angular momentum L = I × ω  (along spin axis)
Applied torque τ → response = τ × L (cross product) → 90° ahead

CCW rotor from above:
  Want disc to tilt FORWARD → apply max pitch at RIGHT side
  Want disc to tilt RIGHT   → apply max pitch at FRONT
```

The swash plate geometry has a built-in 90° phase offset to account for this. On flybarless (electronic) FC helis, this is the `H_PHANG` parameter in ArduPilot — getting it wrong makes controls feel diagonal or reversed.

---

## 8. Headspeed, Governor & Pitch Curve

**Why constant headspeed?** Lift ∝ v² (tip speed). Variable RPM changes lift on all blades simultaneously — uncontrollable. Fixed RPM means only pitch controls lift — clean, linear.

**Governor loop:**
```
Target RPM → compare with actual RPM (from tach/back-EMF)
→ error → PID → throttle adjustment → motor power → RPM correction
```
With a **throttle feedforward** (more collective = more load = pre-estimate more throttle), the governor doesn't lag on aggressive collective inputs.

**Pitch curve** — maps collective stick (0–100%) to blade pitch (degrees):
```
0%  stick → −2°   (slight negative — idle/autorotation)
50% stick → +5°   (hover pitch)
100% stick → +12° (maximum pitch)
```

| Parameter | Typical range |
|-----------|--------------|
| Headspeed | 1500–3000 RPM |
| Blade pitch (collective) | −5° to +12° |
| Cyclic pitch range | ±8° |
| Gear ratio (motor→rotor) | 8:1 to 12:1 |
| Tail rotor ratio | ~4–5× main rotor |

---

## 9. Blade Flapping & Dissymmetry of Lift

![Dissymmetry of lift — FAA diagram](https://commons.wikimedia.org/wiki/Special:FilePath/FAA_heli-manual_Dissymmetry_of_lift.PNG)
*Source: Wikimedia Commons / FAA Helicopter Flying Handbook — Dissymmetry of lift*

In forward flight:
```
Advancing blade: v_tip + v_forward → more lift → flaps UP
Retreating blade: v_tip − v_forward → less lift → flaps DOWN
```

Without flapping, forward flight would roll the helicopter toward the retreating side every time. Flapping equalises the lift automatically.

**Due to precession, the disc tilt peaks 90° after the lift differential** — this is the automatic self-correcting response.

| Effect | Detail |
|--------|--------|
| **Coning angle** | Blades cone upward in hover: centrifugal force (outward) vs lift (upward) balance |
| **Retreating blade stall** | At very high speed, retreating blade must pitch steeply to match lift → stalls → roll/pitch upset — sets helicopter speed limit |
| **δ₃ hinge** | Flapping causes automatic feathering (pitch decrease) — damps rotor passively |

---

## 10. Aerodynamic Effects

### 10.1 Ground Effect

![Ground effect helicopter](https://commons.wikimedia.org/wiki/Special:FilePath/Ground_effect_heli.png)
*Source: Wikimedia Commons — Helicopter ground effect*

When hovering within ~1 rotor diameter of the ground, downwash is blocked → less induced velocity → **10–20% less power for same thrust**.

| | IGE | OGE |
|-|-----|-----|
| Altitude | < 1 rotor diameter | > 1 rotor diameter |
| Power | Less | More |
| ESC sizing | — | **Design for OGE (worst case)** |

As the helicopter climbs out of IGE → OGE, collective must increase — the altitude PID handles this.

---

### 10.2 Vortex Ring State (Settling with Power)

![Vortex ring state](https://commons.wikimedia.org/wiki/Special:FilePath/Vortex_ring_helicopter.jpg)
*Source: Wikimedia Commons — Vortex ring state*

**Condition:** Vertical descent ~300–500 fpm with power applied → helicopter descends into its own downwash → tip vortices recirculate → lift collapses.

```
Rotor ingests its own turbulent wake → chaotic angle of attack → sudden loss of lift
```

- Adding more collective **worsens** it (more vortex energy)
- **Recovery:** forward cyclic — fly out of the vortex ring horizontally
- **FC:** descent rate limit (`PILOT_VELZ_MAX`) prevents entry; barometer monitors descent rate

---

### 10.3 Translational Lift

At ~15–25 km/h forward speed, rotor exits its own downwash → fresh air → **10–15% more lift efficiency**:
- Helicopter pitches up and climbs slightly (FC must compensate)
- Tail rotor load decreases → yaw kick → FC feedforward adjusts tail pitch

---

### 10.4 Autorotation

![Autorotation airflow diagram](https://commons.wikimedia.org/wiki/Special:FilePath/Airflow_in_Autorotation_HEB.jpg)
*Source: Wikimedia Commons — Airflow in autorotation*

Engine-off safe landing: air flows **up** through the descending rotor, spinning the blades like a windmill → RPM maintained without engine.

```
Entry:     Lower collective → pitch to ~0° → rotor maintains RPM via upward airflow
Glide:     Stable descent at ~500–1500 fpm; rotor spins freely
Flare:     Near ground: aft cyclic → disc tilts back → forward speed → lift → slows descent
Touchdown: Pull collective → use stored rotor KE for final cushion
```

**FC implication:** RPM sensor must work with motor power off; FC detects failure (<100 ms) → commands autorotation pitch curve → navigates to landing zone.

---

## 11. Stability

Single-rotor helicopters are **inherently unstable** — no passive restoring force in hover.

| Property | Effect | FC response |
|----------|--------|-------------|
| **Inherently unstable** | Disturbances grow without correction | FC must run ≥400 Hz continuously |
| **Cross-coupling** | Collective → torque → yaw; cyclic → attitude → position | Feedforward decoupling in PID |
| **Pendulum effect** | CG below hub = stable; above hub = unstable | Battery as low as possible |
| **Phase lag** | Servo + swash + blade response ≈ 20–40 ms total | Limits max PID gain |
| **Collective–yaw coupling** | More collective → more torque → yaw disturbance | Collective-to-tail feedforward |

---

## 12. Vibration Sources

At headspeed N RPM with n blades:

| Source | Frequency | At 2000 RPM, 2-blade |
|--------|-----------|----------------------|
| 1P — main imbalance | N/60 | **33 Hz** |
| 2P — blade pass | n × N/60 | **67 Hz** |
| Tail rotor | ratio × N/60 | **~150–167 Hz** |
| Gearbox mesh | teeth × N/60 | Broadband |

**Mitigation:**
- Soft-mount IMU (foam/grommets)
- ArduPilot harmonic notch: `INS_HNTCH_MODE = 3` (tracks RPM, moves with headspeed)
- IMU at board CoM; short solder joints (33–67 Hz cyclic → fatigue cracking)

---

## 13. Servo Requirements

| Parameter | Spec | Why |
|-----------|------|-----|
| Channels | **4 min** (3 CCPM + 1 tail) | Full 4-axis control |
| PWM freq | **400 Hz** | Low latency for attitude loop |
| Speed | **< 0.08 s/60°** | Must respond within one FC cycle |
| Voltage | **6–8.4 V** (HV servos) | More torque + speed vs 5 V standard |
| Signal | **Hardware timer, synchronised** | Jitter > 1 µs → swash wobble |

---

## 14. ESC Requirements

| Parameter | Helicopter | Multirotor |
|-----------|-----------|-----------|
| Motor count | **1** main | 4/6/8 |
| Current | High sustained | High transient |
| Governor | **Required** | Not used |
| BEC | **≥10 A** (4 servos) | ~2 A (FC only) |
| RPM sense | **Required** | Not needed |

**BEC sizing:** 3 CCPM servos peak (6–9 A) + tail (1–2 A) + FC (1 A) = **~10–12 A peak**. Use a dedicated switching BEC — not a linear regulator (too much heat).

---

## 15. AIO PCB Design Checklist

### Power
- [ ] Main ESC stage: single high-current FET bridge — sized for OGE hover current + 30% margin
- [ ] Dedicated switching BEC for servo rail: **6–8.4 V, ≥10 A continuous, ≥15 A peak**
- [ ] Servo BEC output isolated from FC logic supply (separate LDO or filter)
- [ ] Current sensing on main ESC output (for governor load estimation)

### RPM & Governor
- [ ] RPM sense input: tachometer pulse, **3.3 V tolerant**, RC low-pass filter before MCU pin
- [ ] Governor PID in ESC firmware or FC (ArduPilot `H_RSC_MODE = 3` for governor mode)

### Attitude Sensing
- [ ] **Dual IMU** (e.g. ICM-42688-P primary + ICM-20689 backup) — fault detection + redundancy
- [ ] IMU placed at board CoM, **soft-mounted** (foam/grommets)
- [ ] Magnetometer (e.g. IST8310 or QMC5883L) — **≥20 mm from power traces, FETs, and inductors**
- [ ] Barometer (e.g. MS5611, BMP388) for altitude + descent rate estimation (VRS prevention)

### Servo Output
- [ ] **≥4 hardware PWM timer channels** (3 CCPM + 1 tail) — all on the **same timer peripheral** (synchronised)
- [ ] 16-bit timer for <1 µs jitter
- [ ] Optional: CAN/S.Bus/DShot for digital servo communication

### MCU & Communication
- [ ] STM32H7 or F7 — hardware FPU mandatory (quaternion math + CCPM mixing at 400 Hz)
- [ ] ≥256 KB RAM for EKF state matrices, telemetry buffers, PID history
- [ ] MAVLink-compatible UART for ArduPilot integration
- [ ] Dual UART: telemetry + GPS

### Safety
- [ ] Hardware watchdog (IWDG) — resets MCU on hang; servo outputs go to safe position
- [ ] Collective limit in firmware — prevent over-pitch at high headspeed (blade structural limit)
- [ ] Descent rate limit (`PILOT_VELZ_MAX`) — VRS prevention in autopilot modes

---

## Sources
- [01_swash_plate.md](./01_swash_plate.md)
- [02_rotation_orientation_control.md](./02_rotation_orientation_control.md)
- [03_drone_sensors.md](./03_drone_sensors.md)
- NPTEL: Drone Systems and Control, IISc Bangalore
- FAA Helicopter Flying Handbook (images via Wikimedia Commons)
