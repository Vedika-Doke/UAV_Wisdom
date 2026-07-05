# Helicopter — Key Concepts Reference

> Focus: single-rotor helicopter drones with swash-plate control, aimed at AIO FC+ESC PCB design.

---

## Anatomy

```
         ┌──────────────────────────────┐
         │        Main Rotor Blades     │  ← generate lift + thrust vector
         └──────────────┬───────────────┘
                        │ pitch links
         ┌──────────────┴───────────────┐
         │   Rotating Swash Plate       │  ← spins with rotor shaft
         │   Fixed Swash Plate          │  ← connected to servos (static)
         └──────────────┬───────────────┘
                        │ rotor shaft
         ┌──────────────┴───────────────┐
         │   Gearbox / Belt Drive       │  ← steps down motor RPM → rotor RPM
         └──────────────┬───────────────┘
         ┌──────────────┴───────────────┐
         │   Main Brushless Motor       │  ← single high-current drive
         └──────────────────────────────┘

  Fuselage → Tail Boom → Tail Rotor  ← anti-torque + yaw
```

| Component | Role |
|-----------|------|
| Main rotor blades | Generate lift; pitch varies via swash plate |
| Swash plate (fixed + rotating) | Transfers servo commands to spinning blades |
| Pitch links | Rod connecting rotating swash plate to each blade grip |
| Collective servo | Slides swash plate up/down flat → all blades same pitch |
| Cyclic servos ×2 | Tilt swash plate → sinusoidal pitch variation per revolution |
| Tail rotor | Cancels main rotor reaction torque; yaw control |
| Flybar (passive heli) | Gyroscopic stabiliser bar — damps attitude disturbances mechanically |
| Governor / ESC | Holds constant headspeed regardless of collective load |

---

## Rotor Head Types

| Type | Description | Used in |
|------|-------------|---------|
| **Fully articulated** | Each blade has flap, lead-lag, and feathering hinges | Large helicopters; complex |
| **Semi-rigid / Teetering** | Two blades on a common teetering hinge; flap together | Small RC/drone helis; simple |
| **Rigid (hingeless)** | No flap/lag hinges; blade flexibility absorbs forces | Modern drones; stiff, fast response |
| **Bearingless** | Composite flexbeam replaces all hinges | High-performance; minimal maintenance |

> Most small autonomous helicopter drones use **semi-rigid or rigid** heads — simpler, lighter, and faster servo response for the attitude loop.

---

## Control Inputs

| Input | Mechanism | Flight effect |
|-------|-----------|---------------|
| **Collective** | Swash slides up/down (flat) | All blade pitch increases/decreases equally → altitude |
| **Longitudinal Cyclic** | Swash tilts fore/aft | Disc tilts → pitch (nose up/down) + forward/back |
| **Lateral Cyclic** | Swash tilts left/right | Disc tilts → roll + left/right translation |
| **Pedal (Yaw)** | Tail rotor blade pitch changes | More/less anti-torque force → yaw |

> 4 control axes, minimum **4 servo channels** on the AIO FC: 3 swash + 1 tail.

---

## CCPM — Cyclic/Collective Pitch Mixing

Most modern helis use **CCPM (Cyclic Collective Pitch Mixing)** — all 3 swash servos move simultaneously for every input (collective, roll, pitch). The FC firmware does the mixing in software.

```
Servo 1 output = Collective + Pitch_cyclic × sin(120°) + Roll_cyclic × cos(120°)
Servo 2 output = Collective + Pitch_cyclic × sin(240°) + Roll_cyclic × cos(240°)
Servo 3 output = Collective + Pitch_cyclic × sin(0°)   + Roll_cyclic × cos(0°)
```

- 120° CCPM: servos placed 120° apart around the swash plate (most common)
- 90° CCPM: two cyclic + one collective servo (simpler, less symmetrical)

> **AIO PCB:** The mixing equations run on the FC MCU every control cycle. All 3 servo PWM outputs must be time-synchronised (same hardware timer peripheral).

---

## Torque & Anti-Torque

Newton's 3rd law: motor spins rotor → **equal and opposite torque** tries to spin the fuselage the other way.

- **Tail rotor thrust** × tail boom length = moment that cancels main rotor torque
- In hover: tail rotor pitch tuned to exactly balance → zero yaw rate
- More collective (more thrust, more torque) → FC must automatically increase tail rotor pitch to compensate → **collective–yaw coupling** in PID

---

## Gyroscopic Precession

A spinning disc responds to an applied force **90° later** in the direction of rotation.

```
Desired disc tilt direction → apply force 90° ahead (in rotation direction)
```

- Want disc to tilt forward → max blade pitch on the right side (for CCW rotor from above)
- The swash plate geometry/phase is set mechanically to account for this
- For **electronic helicopters (flybarless):** the FC firmware adds a phase compensation angle, configurable per frame

---

## Headspeed & Pitch Curve

**Headspeed** = main rotor RPM. In pitch-controlled helis, headspeed is held **constant** by the governor; altitude is controlled by blade pitch, not RPM.

| Parameter | Typical range (small drone heli) |
|-----------|----------------------------------|
| Headspeed | 1500–3000 RPM |
| Blade pitch range | −5° to +12° (collective travel) |
| Cyclic pitch range | ±8° |
| Gear ratio | 8:1 to 12:1 (motor → main rotor) |
| Tail rotor ratio | ~4–5× main rotor speed |

**Pitch curve:** maps throttle stick (or FC collective command) to blade pitch angle. Allows the FC to command a specific pitch without worrying about mechanical travel limits.

---

## Blade Flapping

Blades are hinged (or flex) to flap up and down in response to unequal lift:

| Blade position | Airspeed seen | Lift | Flap direction |
|----------------|--------------|------|----------------|
| Advancing (into wind) | Higher | More | Up |
| Retreating | Lower | Less | Down |

- **Effect:** disc tilts opposite to advance direction — corrected by cyclic input
- **Coning angle:** blades cone upward in hover due to centrifugal + lift balance; increases with thrust
- **Flapping-to-feathering coupling (δ₃ hinge):** used to reduce pitch-flap coupling, stabilises the rotor

---

## Aerodynamic Effects

### Ground Effect
When hovering close to the ground (<1 rotor diameter altitude), the rotor downwash is partially blocked → reduced induced velocity → **more efficient lift** (less power needed for same thrust).

- IGE (In Ground Effect): easier to hover, more lift margin
- OGE (Out of Ground Effect): true hover power requirement; design the ESC for this

### Vortex Ring State (Settling with Power)
Dangerous condition when descending vertically at moderate rate into the rotor's own downwash:

```
Descent rate ~300–500 fpm + power applied → rotor recirculates tip vortices
→ dramatic loss of lift → rapid uncontrolled descent
```

- Recovery: apply forward cyclic to fly out of the vortex ring
- **FC implication:** descent rate limiting in autopilot firmware prevents entering VRS; barometer + optical flow used to detect and abort

### Translational Lift
At ~15–25 km/h forward speed, the rotor exits its own downwash → sudden efficiency gain:
- Lift increases for same collective pitch
- Yaw kicks as tail rotor load changes (less anti-torque needed)
- FC must anticipate and compensate automatically

---

## Stability Characteristics

| Property | Detail |
|----------|--------|
| **Inherently unstable** | No passive restoring moment in hover — FC must close the loop at all times |
| **Cross-coupling** | Collective → torque → yaw; cyclic → attitude → position; all axes interact |
| **Pendulum effect** | CG below rotor hub increases stability; CG above hub → unstable |
| **Blade lead-lag** | Blades also hinge in-plane (Coriolis forces); adds vibration at N×RPM |
| **Phase lag** | Servo + swash plate + blade response introduces lag into attitude loop; limits max PID gain |

---

## Vibration Sources

| Source | Frequency formula | Typical (2000 RPM, 2-blade) |
|--------|--------------------|------------------------------|
| 1P (once per rev) | RPM / 60 | ~33 Hz |
| 2P blade pass | 2 × RPM / 60 | ~67 Hz |
| Tail rotor (nP) | Tail ratio × RPM / 60 | ~150–200 Hz |
| Gearbox mesh | Gear teeth × RPM / 60 | broadband |

**Mitigation for AIO PCB:**
- Soft-mount the FC board on foam/rubber grommets
- Notch filters at 1P and 2P in FC firmware (ArduPilot: INS_HNTCH_FREQ)
- Place IMU at CoM, away from motor mount
- Keep solder joints short and supported — cyclic vibration causes fatigue cracking

---

## Servo Requirements

| Parameter | Value | Reason |
|-----------|-------|--------|
| Channels | 4 minimum (3 CCPM + 1 tail) | Full 4-axis control |
| Signal | PWM 400 Hz or digital (HV servo bus) | Low latency for attitude loop |
| Torque | 5–15 kg·cm (scale dependent) | Cyclic loads are dynamic |
| Speed | < 0.08 s / 60° | Fast enough for >100 Hz attitude loop |
| Voltage | 6–8.4 V (HV servos) | Higher V = more torque + speed |
| PWM resolution | 12-bit minimum | Fine pitch granularity |

> Use **hardware timer channels** on the MCU for PWM — never GPIO bit-bang. Jitter > 1 µs causes audible and mechanical servo hunting.

---

## ESC Requirements

| Parameter | Helicopter | Multirotor |
|-----------|-----------|-----------|
| Motor count driven | 1 (main) | 4/6/8 |
| Current profile | High sustained, low transient variation | High transient (rapid throttle changes) |
| Governor | Required | Not used |
| BEC output | Must support ≥4 servos (≥10 A total) | Minimal (FC only) |
| RPM sense | Tachometer pulse or back-EMF zero-cross | Not needed |
| Switching freq | 16–32 kHz (low audible noise preferred) | 24–48 kHz |

**BEC note:** Servo current spikes during rapid cyclic inputs. Use a dedicated BEC with ≥10 A continuous, or separate servo power rail from main ESC BEC to avoid voltage droop causing FC brownout.

---

## AIO PCB Design Checklist

- [ ] ≥4 hardware PWM timer channels (3 CCPM servos + 1 tail)
- [ ] RPM sense input (tachometer pulse, 3.3 V tolerant)
- [ ] Dual IMU (primary + backup), soft-mounted at CoM
- [ ] Magnetometer away from motor power traces and inductors
- [ ] Dedicated servo BEC rail (6–8.4 V, ≥10 A), isolated from FC logic supply
- [ ] Main ESC stage: single high-current FET bridge, governor firmware
- [ ] Barometer for altitude + descent rate (VRS prevention)
- [ ] MCU with FPU (STM32H7 / F7) — quaternion math + CCPM mixing every 400 Hz cycle
- [ ] CAN or UART telemetry for governor status feedback to autopilot

---

## Sources
- [01_swash_plate.md](./01_swash_plate.md) — swash plate mechanics
- [02_rotation_orientation_control.md](./02_rotation_orientation_control.md) — PID loops, flight control architecture
- NPTEL: Drone Systems and Control, IISc Bangalore
