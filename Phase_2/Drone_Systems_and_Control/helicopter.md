# Helicopter — Key Concepts Reference

> Focus: single-rotor helicopter drones with swash-plate control. Relevant to AIO FC+ESC PCB design.

---

## Anatomy

```
              ┌─────────────────┐
              │   Main Rotor    │  ← generates lift + thrust vector
              └────────┬────────┘
                       │ rotor shaft
              ┌────────┴────────┐
              │   Swash Plate   │  ← translates servo inputs to blade pitch
              └────────┬────────┘
              ┌────────┴────────┐
              │   Gearbox /     │  ← steps down motor RPM to rotor RPM
              │   Belt Drive    │
              └────────┬────────┘
              ┌────────┴────────┐
              │  Brushless      │  ← single high-current motor (main)
              │  Motor (main)   │
              └─────────────────┘

  Tail boom → Tail Rotor → anti-torque + yaw control
```

| Component | Role |
|-----------|------|
| Main rotor blades | Generate lift via angle of attack |
| Swash plate (fixed + rotating) | Interface between static servos and spinning blades |
| Pitch links | Connect rotating swash plate to individual blade grips |
| Collective servo | Slides swash plate up/down → changes all blade pitch equally |
| Cyclic servos (×2) | Tilt swash plate → sinusoidal pitch variation per revolution |
| Tail rotor | Counter-acts main rotor torque; yaw control via pitch change |
| Flybar (if present) | Passive stabiliser — gyroscopic bar that damps attitude disturbances |
| Governor / ESC | Maintains constant main rotor RPM regardless of load |

---

## Control Inputs

| Input | Mechanism | Effect |
|-------|-----------|--------|
| **Collective** | Swash plate slides up/down (flat) | All blades change pitch equally → altitude control |
| **Longitudinal Cyclic** | Swash plate tilts fore/aft | Disc tilts → forward/back translation |
| **Lateral Cyclic** | Swash plate tilts left/right | Disc tilts → left/right translation |
| **Pedal (Yaw)** | Tail rotor blade pitch | More/less anti-torque → yaw rotation |

> In RC/autonomous heli: collective servo + 2 cyclic servos + tail servo = **4 servo channels minimum.**

---

## Torque & Anti-Torque

Newton's 3rd law: the motor spins the rotor one way → **equal and opposite torque** tries to spin the body the other way.

- **Main rotor torque** → body wants to rotate opposite to blade direction
- **Tail rotor** applies a sideways thrust force at the tail boom → cancels the yaw torque
- In hover: tail rotor pitch is set to exactly cancel main rotor torque
- Increase throttle (more main rotor torque) → must increase tail rotor pitch too — this is why **collective and tail rotor are coupled** in the PID

---

## Gyroscopic Precession

A spinning disc responds to an applied force **90° later** in the direction of rotation.

- If you want the disc to tilt forward → you must apply the force at the **right side** (90° ahead)
- The swash plate accounts for this: max blade pitch is commanded 90° before the desired disc tilt point
- **Why it matters for the FC:** the control allocation algorithm must bake in this phase advance; raw cyclic commands are not in the same frame as desired disc tilt without compensation

---

## Blade Flapping

Blades are not rigidly fixed — they are hinged (or flexible) to flap up and down:

| Situation | What happens |
|-----------|-------------|
| Advancing blade (moving into airflow) | Sees higher airspeed → more lift → flaps up |
| Retreating blade | Sees lower airspeed → less lift → flaps down |
| Result | Disc tilts naturally (flapping equality) |

- Flapping relieves asymmetric lift in forward flight (prevents roll)
- The swash plate tilt compensates for this to achieve intended disc tilt

---

## RPM & Governor

Main rotor RPM is fixed (unlike multirotors where RPM varies for control). Altitude is controlled by **blade pitch, not RPM**.

| Mode | How |
|------|-----|
| **Governor ON** | ESC measures RPM (via tachometer or back-EMF) and actively adjusts throttle to hold constant RPM |
| **Governor OFF** (throttle curve) | Pilot/FC sets throttle curve; RPM varies with load |

> **AIO PCB:** ESC must expose an RPM sense pin (or use back-EMF zero-cross detection). The FC reads this to verify rotor is at headspeed before allowing control authority.

---

## Flight Phases

| Phase | Key dynamics |
|-------|-------------|
| **Hover** | Thrust = weight; tail rotor cancels torque; PID holds attitude |
| **Forward flight** | Longitudinal cyclic tilts disc forward; body pitches nose-down; airspeed builds |
| **Autorotation** | Engine off; collective lowered → blades windmill on descending air; energy stored in rotor inertia; collective pulled at flare → controlled landing |
| **Translational lift** | ~15–25 km/h forward speed; rotor exits its own downwash → efficiency spike; noticeable yaw kick as tail rotor load changes |

---

## Stability Characteristics

- **Inherently unstable in hover** — unlike a fixed-wing, a helicopter has no passive restoring moment; the FC must actively stabilise at all times
- **Cross-coupling:** collective input changes torque → yaw changes; cyclic input changes attitude → position changes. The FC must decouple these
- **Blade lead-lag:** blades also hinge in the plane of rotation to absorb Coriolis forces from flapping — adds vibration at blade-pass frequency

---

## Vibration Sources (critical for AIO PCB)

| Source | Frequency | Impact |
|--------|-----------|--------|
| Main rotor (1P) | RPM / 60 Hz | Low-freq; dominant |
| Blade pass (2P or 3P) | N_blades × RPM / 60 | IMU noise |
| Tail rotor | ~4–6× main rotor RPM | High-freq buzz |
| Gearbox meshing | Gear ratio × RPM | Broadband |

→ IMU must be **vibration-isolated** (soft mount / foam dampers). Notch filters in FC firmware target 1P and 2P frequencies.

---

## Servo Requirements (AIO PCB)

| Parameter | Typical value | Note |
|-----------|--------------|------|
| Channels needed | 4 (3 swash + 1 tail) | Minimum for full control |
| Signal type | PWM 50 Hz / Digital (SBUS, PWM 400 Hz) | Higher rate = lower latency |
| Torque | 5–15 kg·cm (scale dependent) | Swash servos see cyclic loads |
| Speed | < 0.10 s / 60° | Fast response critical for attitude loop |
| PWM resolution | 12-bit minimum | Fine pitch control |

> Use **hardware timers** for PWM generation on the MCU — bit-banged GPIO introduces jitter that degrades cyclic control precision.

---

## ESC Requirements (AIO PCB)

| Parameter | Helicopter vs Multirotor |
|-----------|--------------------------|
| Motor count | 1 main + 1 tail (or fixed-pitch tail) |
| Current profile | High sustained; relatively steady (not rapid switching) |
| Governor | Required — RPM feedback loop built into ESC |
| BEC | Must power 4 servos; dedicated BEC preferred (not shared with main power) |
| RPM sense | Tachometer input or back-EMF detection output |

---

## Sources
- [01_swash_plate.md](./01_swash_plate.md) — swash plate mechanics in detail
- [02_rotation_orientation_control.md](./02_rotation_orientation_control.md) — PID loops and flight control architecture
- NPTEL: Drone Systems and Control, IISc Bangalore
