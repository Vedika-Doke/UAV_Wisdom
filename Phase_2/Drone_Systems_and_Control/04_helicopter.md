# Helicopter — Key Concepts Reference

> Focus: single-rotor helicopter drones with swash-plate control, aimed at AIO FC+ESC PCB design.  
> This document explains **why** each concept works, not just what it is.

---

## 1. Anatomy

```
         ┌──────────────────────────────────────┐
         │         Main Rotor Blades            │  ← generate lift + control thrust vector
         └──────────────────┬───────────────────┘
                            │ pitch links (push-rods)
         ┌──────────────────┴───────────────────┐
         │        Rotating Swash Plate          │  ← spins with rotor shaft
         │        Fixed Swash Plate             │  ← connected to servos (static frame)
         └──────────────────┬───────────────────┘
                            │ rotor shaft
         ┌──────────────────┴───────────────────┐
         │         Gearbox / Belt Drive         │  ← steps down motor RPM → rotor RPM
         └──────────────────┬───────────────────┘
         ┌──────────────────┴───────────────────┐
         │        Main Brushless Motor          │  ← single high-current drive
         └──────────────────────────────────────┘

  Fuselage → Tail Boom ──────────────────────→ Tail Rotor
                                                ↑ anti-torque + yaw control
```

| Component | Role | PCB implication |
|-----------|------|----------------|
| Main rotor blades | Generate lift; pitch varies via swash plate | — |
| Swash plate (fixed + rotating) | Transfers servo commands to spinning blades | Servo timing must be synchronised |
| Pitch links | Push-rod connecting rotating swash plate to each blade grip | Mechanical; link length sets pitch range |
| Collective servo | Slides swash plate up/down flat → all blades same pitch | One of 3 CCPM servos |
| Cyclic servos ×2 | Tilt swash plate → sinusoidal pitch variation per revolution | Must be hardware-timed |
| Tail rotor | Cancels main rotor reaction torque; provides yaw control | 1 servo channel for tail pitch |
| Flybar (passive heli only) | Gyroscopic stabiliser bar — damps attitude disturbances mechanically | Not present in flybarless |
| Governor / ESC | Holds constant headspeed regardless of collective load | Must read RPM feedback on AIO board |

---

## 2. How Lift Is Generated

Understanding lift is fundamental before understanding control.

**The lift equation:**
```
L = ½ · ρ · v² · A · C_L(α)

where:
  ρ    = air density (kg/m³) — decreases with altitude, affects all performance
  v    = blade tip speed (m/s) = ω × r = (2π × RPM/60) × blade_radius
  A    = blade disc area (m²)
  C_L  = lift coefficient — depends on angle of attack (α) and blade profile
  α    = angle of attack = geometric pitch − induced inflow angle
```

**Key insight — helicopters control lift via pitch, not RPM:**
- Multirotor: v (RPM) changes → lift changes
- Helicopter: RPM is **constant** (governed); blade pitch angle α changes → C_L changes → lift changes
- This gives much faster lift response (changing blade pitch is nearly instantaneous; spinning up/down a motor takes 100–500 ms)

**Blade tip speed** is the dominant term (v²). For a typical small drone helicopter:
```
RPM = 2000, blade radius = 0.5 m
v_tip = (2π × 2000/60) × 0.5 ≈ 105 m/s  (about Mach 0.3)
```
This is why blade flex, mass balance, and tracking are critical — at 105 m/s, a 1 mm tracking error causes significant vibration.

---

## 3. Rotor Head Types

| Type | Hinges present | Blade motion absorbed by | Response | Used in |
|------|---------------|--------------------------|----------|---------|
| **Fully articulated** | Flap + lead-lag + feather | Discrete hinges | Smooth, slow | Large manned helicopters |
| **Semi-rigid / Teetering** | One teetering hinge (2-blade) | Both blades flap together | Medium | Small RC/drone helis |
| **Rigid (hingeless)** | None | Blade root flexibility | Fast, stiff | Modern drone helis |
| **Bearingless** | None | Composite flexbeam | Very fast | High-performance |

**Why rigid/semi-rigid for drones:**
- Fewer parts → less weight, less maintenance
- No hinge friction → servo commands propagate directly to blade pitch
- Faster attitude loop response (critical for flybarless FC-controlled helis)
- Downside: vibration transmitted directly to airframe (must soft-mount IMU)

**Feathering** = blade pitch change (the useful one, controlled by swash plate).  
**Flapping** = blade up/down motion (aerodynamic response to unequal lift — see §9).  
**Lead-lag** = blade fore/aft motion (Coriolis response to flapping — creates vibration).

---

## 4. Control Inputs

A helicopter has **4 independent control axes**, all via blade pitch:

| Input | Physical mechanism | Primary flight effect | Secondary effect |
|-------|-------------------|----------------------|------------------|
| **Collective** | Swash plate slides straight up/down (no tilt) | All blades same pitch → total lift → altitude | More torque → yaw coupling |
| **Longitudinal Cyclic** | Swash plate tilts fore/aft | Disc tilts → forward/back | Slight altitude change |
| **Lateral Cyclic** | Swash plate tilts left/right | Disc tilts → left/right | Slight altitude change |
| **Pedal (Yaw)** | Tail rotor blade pitch changes | Anti-torque force → yaw | Slight roll from tail thrust |

**How cyclic works (intuition):**
The swash plate tilt causes each blade's pitch to vary sinusoidally as it goes around — high pitch on one side, low pitch on the other. This creates more lift on one side → disc tilts that way → helicopter moves that way.

```
Swash plate tilted forward:
  Blade at 180° (rear) → high pitch → more lift → blade flaps up (90° later = left side)
  Blade at 0° (front)  → low pitch → less lift → blade flaps down (90° later = right side)
  Net: disc tilts forward → helicopter pitches forward → moves forward
```

> The 90° delay between applied pitch and disc response is **gyroscopic precession** (§7).

**Why you need 4 channels minimum on the AIO FC:**
```
Channel 1: Swash servo A  ─┐
Channel 2: Swash servo B  ─┤─ All 3 move together for any collective/cyclic input (CCPM)
Channel 3: Swash servo C  ─┘
Channel 4: Tail rotor servo
```

---

## 5. CCPM — Cyclic/Collective Pitch Mixing

**CCPM (Cyclic Collective Pitch Mixing):** All 3 swash servos move simultaneously for every control input. The FC does the math in firmware every control cycle.

### 120° CCPM Mixing Equations

```
Servo 1 (at   0°): output = Collective + Pitch × sin(  0°) + Roll × cos(  0°)
Servo 2 (at 120°): output = Collective + Pitch × sin(120°) + Roll × cos(120°)
Servo 3 (at 240°): output = Collective + Pitch × sin(240°) + Roll × cos(240°)
```

Substituting values (sin120° = 0.866, cos120° = −0.5, sin240° = −0.866, cos240° = −0.5):

```
Servo 1: output = Collective +   0    × Pitch + 1.000 × Roll
Servo 2: output = Collective + 0.866  × Pitch − 0.500 × Roll
Servo 3: output = Collective − 0.866  × Pitch − 0.500 × Roll
```

**Worked example:** Collective = 1500 µs (centre), Pitch command = +200 µs, Roll = 0:
```
Servo 1: 1500 + 0      + 0   = 1500 µs  (no change — this servo is at 0°, perpendicular to pitch)
Servo 2: 1500 + 173.2  + 0   = 1673 µs  (moves up)
Servo 3: 1500 − 173.2  + 0   = 1327 µs  (moves down)
→ Swash plate tilts in the pitch direction ✓
```

**Pure collective (altitude, no tilt):** Pitch = 0, Roll = 0 → all 3 servos move equally by `Collective` amount → swash plate rises/falls flat.

### Why CCPM over traditional mixing?

| Traditional (1 collective + 2 cyclic servos) | CCPM (3 identical servos) |
|---------------------------------------------|--------------------------|
| Servos have different tasks | All servos share load equally |
| Collective servo handles full collective load alone | Load distributed across 3 |
| Asymmetric loading → different wear | Symmetric → more reliable |
| Less efficient mechanically | More efficient |

> **AIO PCB:** Mixing runs on the MCU every attitude cycle (400 Hz). All 3 servo PWM signals must be generated by the **same hardware timer peripheral** to guarantee synchronised rising edges — a skew of even 10 µs causes a perceptible swash plate wobble.

---

## 6. Torque & Anti-Torque

**Why torque is a problem:**

Newton's 3rd law: the motor applies torque to spin the rotor. The rotor applies an equal and opposite torque back on the fuselage. Without correction, the helicopter body spins opposite to the rotor.

```
Main rotor torque (CCW from above) → fuselage reaction = CW rotation

Anti-torque moment = F_tail × L_tail_boom

where:
  F_tail = thrust from tail rotor (N)
  L_tail = distance from main rotor shaft to tail rotor (m)
```

**Hover equilibrium:**
```
F_tail × L_tail = Q_main_rotor

Q_main (Newton-metres) scales with collective pitch → as you pull collective, torque increases → tail rotor must work harder → **more tail pitch** needed automatically
```

This coupling is why the FC has a **collective-to-yaw feedforward**: when the collective PID commands up, it simultaneously adds a proportional offset to the tail rotor command, before the yaw error even develops.

### Tail Rotor Configurations

| Type | How pitch changes | Advantages | Drawbacks |
|------|------------------|-----------|-----------|
| **Variable pitch** (standard) | Servo changes blade pitch | Fast yaw response; governor keeps RPM | More mechanical complexity |
| **Fixed pitch, variable RPM** (some small drones) | Tail ESC changes RPM | Simpler mechanics | Slow yaw response (~200 ms vs ~30 ms) |
| **Fenestron** | Fan in tail boom shroud | Quieter, safer, more efficient | Heavier, less effective in crosswind |
| **NOTAR** | Directed airflow from tail boom | No exposed rotor — very safe | Complex, heavier |

> For AIO drone FC: variable pitch tail is preferred — the tail servo channel is essential. Fixed-pitch tail (RPM-controlled) would need a second ESC output, adding complexity.

---

## 7. Gyroscopic Precession

**The physics:**

A spinning rotor is a gyroscope. When you apply a torque to tilt a gyroscope, it responds in a direction **90° ahead in the direction of rotation** — not in the direction you applied the torque.

```
Angular momentum vector L = I × ω  (points along spin axis, right-hand rule)

Applied torque τ → rate of change of L:
  dL/dt = τ × L  (cross product)
  → The response is perpendicular to both τ and L
  → Effectively 90° ahead in rotation direction
```

**Practical example (CCW rotor from above):**

```
Want disc to tilt forward (nose down):
  → Must apply max blade pitch at the RIGHT side of the disc
  → That force, processed 90° ahead (in CCW direction), tilts disc FORWARD ✓

Want disc to tilt right:
  → Must apply max blade pitch at the FRONT of the disc
  → Processed 90° ahead → tilts disc RIGHT ✓
```

**How swash plate handles this mechanically:**

The swash plate is designed with a **phase angle offset** (typically 90° for most rotor systems) so that when you push the cyclic stick forward, the swash plate applies maximum pitch at the correct 90°-ahead position. This is set during helicopter design/assembly.

**Flybarless (electronic) helicopters:**

The FC can apply a software phase offset, which is configurable. This is the `H_PHANG` parameter in ArduPilot (Heli pitch angle). Getting this wrong means controls feel reversed or diagonal — the FC compensates incorrectly.

---

## 8. Headspeed, Governor & Pitch Curve

### Why Constant Headspeed?

Lift = ½ρv²AC_L where v = tip speed ∝ RPM. If RPM varies:
- Collective pitch changes to maintain altitude
- But RPM affects ALL lift simultaneously — uncontrollable coupling
- Cyclic authority also changes with RPM (control sensitivity becomes inconsistent)

Holding RPM constant means **only blade pitch controls lift** — clean, linear control for the FC.

### Governor Operation

The governor maintains constant headspeed by adjusting throttle (motor power) as collective pitch changes:

```
More collective → more aerodynamic drag on blades → rotor decelerates
Governor: senses RPM drop → increases throttle → RPM returns to setpoint

Less collective → less drag → rotor accelerates
Governor: senses RPM rise → decreases throttle → RPM stays constant
```

**RPM sensing methods:**
- **Motor back-EMF zero-crossing:** ESC measures it internally; no extra hardware
- **Dedicated tachometer:** Optical or magnetic pulse sensor on rotor shaft — more accurate, requires a 3.3 V digital input on the AIO board

**Governor tuning parameters:**

| Parameter | Effect |
|-----------|--------|
| P gain | How aggressively throttle responds to RPM error |
| I gain | Eliminates steady-state RPM error under load |
| Headspeed setpoint | RPM target (e.g. 2000 RPM) |
| Throttle curve | Initial throttle estimate vs collective — helps governor stay in linear range |

### Pitch Curve

The pitch curve maps collective stick position (0–100%) to blade pitch angle (degrees):

```
Example flat pitch curve (stable hover):
  0%  stick → −2° (slight negative pitch — for inverted autorotation or idle)
  50% stick → +5° (hover pitch — where motor load balances weight)
  100% stick → +12° (maximum positive pitch)

Linear: pitch = −2 + (14/100) × stick_percent
```

A well-tuned pitch curve keeps the helicopter at hover pitch at mid-stick, giving equal authority up and down.

| Parameter | Typical value |
|-----------|--------------|
| Headspeed | 1500–3000 RPM |
| Blade pitch range (collective) | −5° to +12° |
| Cyclic pitch range | ±8° |
| Gear ratio (motor → main rotor) | 8:1 to 12:1 |
| Tail rotor ratio | ~4–5× main rotor speed |

---

## 9. Blade Flapping & Asymmetry of Lift

### Why Flapping Occurs

In forward flight, the advancing blade (moving into the relative wind) has a higher total airspeed, while the retreating blade has a lower total airspeed:

```
Blade airspeed = Rotor tip speed ± Forward flight speed

Advancing blade: v_tip + v_forward → more lift
Retreating blade: v_tip − v_forward → less lift
```

Without flapping, this would roll the helicopter toward the retreating side every time it moved forward. Flapping is the mechanism that equalises this.

| Blade position | Airspeed | Lift | Flap response |
|----------------|----------|------|---------------|
| Advancing (into wind) | Higher | More | Flaps **up** |
| Retreating (with wind) | Lower | Less | Flaps **down** |

**Due to gyroscopic precession, blade flapping peaks 90° after the lift differential** — the disc tilts opposite to the advance direction. This is the automatic flapping response that prevents the uncontrolled roll.

### Coning Angle

In hover, blades do not flap periodically — they cone upward symmetrically:

```
Coning angle β:
  Centrifugal force pulls blades outward (horizontal)
  Lift force pulls blades upward
  Equilibrium cone angle: β = arctan(Lift / Centrifugal force)

Higher thrust → higher β (more upward coning)
```

Too much coning reduces rotor effective disc area and increases stress at the blade root.

### Retreating Blade Stall

At very high forward speed, the retreating blade slows down enough that its angle of attack must increase dramatically to maintain equal lift — eventually it **stalls**:

```
Retreating side: v_tip − v_forward is low
  → to maintain lift, pitch increased
  → angle of attack exceeds α_critical → blade stalls
  → sudden asymmetric lift loss → violent roll/pitch
```

This is the fundamental speed limit of single-rotor helicopters. For small drone helis, this limit is rarely reached.

### δ₃ (Delta-Three) Hinge

The δ₃ hinge tilts the flapping axis slightly so that flapping causes automatic feathering (pitch change):
- When blade flaps up → pitch automatically decreases (less lift) → damps the flapping
- Stabilises the rotor passively without requiring servo correction

---

## 10. Aerodynamic Effects

### 10.1 Ground Effect (IGE vs OGE)

When hovering close to the ground (within ~1 rotor diameter altitude):

**Why it helps:**
The rotor downwash normally flows downward and outward, creating a column of descending air that the rotor must continuously push against (induced velocity). Near the ground, this downwash is blocked — it cannot continue descending and instead spreads outward. This reduces the induced downwash the rotor has to overcome.

```
Less induced velocity → less induced drag → same thrust with less power
Power savings in IGE: typically 10–20% less power than OGE for same thrust
```

| | IGE | OGE |
|-|-----|-----|
| Altitude | < 1 rotor diameter | > 1 rotor diameter |
| Power required | Less | More |
| Hover ceiling | Higher | Lower |
| Design target | — | **Design ESC for OGE** (worst case) |

**FC implication:** When taking off, the helicopter climbs through IGE → OGE transition. Collective must increase slightly as it climbs out of ground effect — the altitude controller handles this automatically via the collective PID.

### 10.2 Vortex Ring State (Settling with Power)

One of the most dangerous helicopter flight conditions.

**How it forms:**

```
Normal hover:
  Rotor pushes air DOWN → clean column of descending flow below the disc

Vertical descent at ~300–500 fpm (1.5–2.5 m/s) with power applied:
  The descending helicopter "catches up" with its own downwash
  → Downwash and fuselage descent velocity cancel near the disc
  → Air recirculates in a toroidal (donut-shaped) vortex around the blade tips
  → Rotor is now ingesting its own turbulent wake
  → Effective angle of attack on blades becomes chaotic
  → Lift collapses → rapid uncontrolled descent
```

**Why "settling with power":** The pilot has power applied (trying to descend slowly) but still descends rapidly — adding more collective makes it worse (increases the vortex energy).

**VRS boundary (approximate):**
```
Descent rate > ~0.3 × v_h  AND  0 < forward speed < ~15 km/h
where v_h = hover-induced velocity ≈ √(T / 2ρA)
```

**Recovery:**
1. Apply forward cyclic aggressively — fly out of the vortex ring horizontally
2. Once clear (~5–10 m forward), re-establish normal descent
3. Do NOT add more collective — worsens VRS

**FC firmware implications:**
- Autopilot firmware (ArduPilot `LAND_SPEED`, `PILOT_VELZ_MAX`) limits descent rate
- Barometer + optical flow velocity estimate used to detect high-rate descent with low forward speed
- Some FC implementations add a feedforward to increase collective slightly during fast descent to stay out of the VRS region

### 10.3 Translational Lift

At approximately 15–25 km/h forward speed, the rotor disc moves forward fast enough to **fly out of its own downwash column**:

```
Hover:    Rotor re-ingests turbulent, recirculated downwash → less efficient
Forward:  Fresh, undisturbed air continuously fed to rotor disc → higher effective angle of attack
```

**Effects:**
1. **Lift increases** for the same collective pitch → helicopter pitches up and climbs slightly
2. **Rotor efficiency improves** by 10–15% (induced velocity decreases)
3. **Tail rotor load changes:** as fuselage moves forward, the tail rotor also gains translational lift → less tail pitch needed to maintain heading → yaw disturbance

**FC handling:**
- Position/velocity controller must apply a slight forward pitch correction as translational lift adds unexpected climb
- Yaw compensation feedforward must account for changing tail rotor efficiency vs airspeed

### 10.4 Autorotation

The ability to land safely with zero engine power — critical safety concept.

**How it works:**

```
Engine fails → rotor starts to slow down
Pilot immediately lowers collective → blade pitch goes to 0° or slightly negative
→ Helicopter descends, air flows UP through the disc (opposite of normal powered flight)
→ This upward airflow drives the rotor blades, maintaining RPM without engine
→ Helicopter autorotates — rotor is now a windmill, not a propeller
```

**Phases:**
1. **Entry:** Lower collective immediately to keep rotor RPM up (inertia of spinning rotor is energy stored)
2. **Glide:** Stable descent at ~500–1500 fpm; rotor RPM maintained by airflow
3. **Flare:** Near ground, pull aft cyclic → increase angle of attack → convert forward speed to lift → reduce descent rate
4. **Touchdown:** Pull collective just before ground → convert stored rotor kinetic energy into final lift → cushion landing

**For autonomous helicopter drones:**
- The FC must detect engine failure (RPM drops below threshold in < 100 ms)
- Immediately command collective to 0° (or autorotation pitch curve)
- Engage autorotation mode — fly to a pre-computed landing site
- Autorotation RPM sensing requires the tachometer input on the AIO board to work even with motor power off

---

## 11. Stability Characteristics

Single-rotor helicopters are **inherently unstable** in hover — there is no passive restoring force. The FC must actively close the loop at all times.

| Property | Physics explanation | FC implication |
|----------|--------------------|-|
| **Inherently unstable** | No passive restoring moment; a small disturbance grows without correction | FC must run at ≥400 Hz; any latency → instability |
| **Cross-coupling** | Collective → more torque → yaw; cyclic → disc tilt → position drift; all axes interact | PID cross-decoupling feedforward terms needed |
| **Pendulum effect** | CG below rotor hub creates restoring moment (like a pendulum); CG above hub → unstable inverted pendulum | Place battery (heaviest component) as low as possible on the frame |
| **Blade lead-lag** | Blades move fore/aft due to Coriolis force as they flap; creates N×RPM vibration | Notch filter at N×RPM in FC firmware |
| **Phase lag** | Servo actuation (~10 ms) + swash plate mechanical response + blade aerodynamic response → total ~20–40 ms lag | Limits maximum PID gain; derivative must account for lag |
| **Collective–yaw coupling** | More collective → more main rotor torque → yaw disturbance | Feedforward: collective command → proportional tail rotor offset |
| **Translational instability** | Moving forward → translational lift → pitch up → decelerates → oscillates | Position + velocity outer loop; airspeed damping |

---

## 12. Vibration Sources & Mitigation

### Vibration Frequencies

At headspeed N (RPM), with n blades:

| Source | Frequency | At 2000 RPM, 2-blade |
|--------|-----------|----------------------|
| 1P — once per revolution (main imbalance) | N/60 | **33.3 Hz** |
| 2P — blade pass frequency | n × N/60 | **66.7 Hz** |
| Tail rotor | Tail_ratio × N/60 | **~150–167 Hz** (at 4.5× ratio) |
| Gearbox mesh | Gear_teeth × N/60 | Broadband |
| Motor electrical | Motor_poles/2 × N/60 | Depends on motor |

**Why 1P is dominant:** Any mass imbalance on the main rotor hub (unequal blade masses, tracking error) creates a once-per-revolution centrifugal excitation. This is felt as a lateral/longitudinal 33 Hz shake at 2000 RPM.

**Why 2P matters:** Even with perfect balance, aerodynamic asymmetries from retreating blade / advancing blade cycle create a 2-per-revolution pitching/rolling moment.

### Mitigation Strategy for AIO PCB

| Layer | Technique | Implementation |
|-------|-----------|---------------|
| **Mechanical** | Track and balance blades | Equal blade weights; adjust tracking via pitch link |
| **Mechanical** | Soft-mount IMU | 2–3 mm foam tape or rubber grommets under IMU or FC board |
| **Firmware** | Notch filter at 1P | ArduPilot: `INS_HNTCH_FREQ = headspeed_hz`, `INS_HNTCH_BW = 5` |
| **Firmware** | Harmonic notch (tracks RPM) | ArduPilot: `INS_HNTCH_MODE = 3` (uses RPM sensor) — filter moves with headspeed |
| **PCB layout** | IMU at CoM | Place IMU chip at board's centre of mass, not edge |
| **PCB layout** | Short solder joints | Cyclic vibration at 33–67 Hz causes fatigue cracking of cold/long joints |
| **PCB layout** | Ground planes continuous | Vibrating board with discontinuous ground causes ADC noise spikes |

---

## 13. Servo Requirements

| Parameter | Specification | Why |
|-----------|--------------|-----|
| Channels | **4 minimum** (3 CCPM + 1 tail) | Full 4-axis control |
| PWM frequency | **400 Hz** (or CAN/digital bus) | Low latency for 400 Hz attitude loop |
| Torque | 5–15 kg·cm (scale-dependent) | Cyclic loads are dynamic and impulsive |
| Speed | **< 0.08 s/60°** at rated voltage | Must respond within one attitude loop cycle |
| Voltage | **6.0–8.4 V** (HV servos) | Higher V = more torque + speed; standard 5V servos too slow |
| PWM resolution | **12-bit minimum** | 1-bit error at 12-bit → 0.02° pitch error; coarser → audible servo hunting |
| Signal timing | **Hardware timer, synchronised** | Jitter > 1 µs causes swash plate wobble and IMU noise |

**Why HV (high-voltage) servos:**

Standard servo: 5V → peak current ~2A → torque ~3 kg·cm  
HV servo: 7.4V → peak current ~3A → torque ~8 kg·cm  
Same response time, double the torque — essential for large blades or high headspeed.

**Why synchronised PWM:**

All 3 CCPM servos must receive their new position command at the same moment. If servo 1 updates 1 ms before servo 2, the swash plate briefly tilts incorrectly, injecting an attitude error that the IMU sees and the PID amplifies.

---

## 14. ESC Requirements & Governor

| Parameter | Helicopter | Multirotor |
|-----------|-----------|-----------|
| Motor count | **1** (main) + 1 tail (small) | 4/6/8 |
| Current profile | **High sustained** (constant headspeed); moderate transients | High transient (rapid throttle changes) |
| Governor | **Required** — holds constant headspeed | Not used |
| BEC output | **≥10 A continuous** (≥4 servos) | Minimal (FC only, ~2 A) |
| RPM feedback | **Required** (tach pulse or back-EMF) | Not needed |
| Switching frequency | **16–32 kHz** (low acoustic noise preferred) | 24–48 kHz |
| Current sensor | Needed for governor load estimation | Optional |

### Governor Detail

The governor is a closed-loop RPM controller running inside the ESC firmware:

```
Target RPM setpoint (e.g. 2000 RPM)
  ↓
RPM sensor → actual RPM
  ↓
Error = setpoint − actual
  ↓
ESC throttle = Throttle_feedforward + P×error + I×∫error dt
  ↓
Motor voltage → rotor RPM changes
```

**Throttle feedforward:** A pre-computed throttle estimate based on current collective command (roughly: more collective = more load = more throttle needed). This prevents the governor from lagging during aggressive collective inputs.

**BEC sizing:**

| Load | Current draw |
|------|-------------|
| 3 CCPM servos at peak (cyclic input) | 3 × 2–3 A = 6–9 A |
| 1 tail servo at peak | 1–2 A |
| FC board | 0.5–1 A |
| **Total peak** | **~10–12 A** |

Use a dedicated switching BEC (not linear — too much heat) rated for ≥10 A continuous, 15 A peak. If co-located with the ESC on the AIO board, route the BEC output on a separate copper pour from the main ESC FET traces to avoid noise coupling.

---

## 15. AIO PCB Design Checklist

### Power
- [ ] Main ESC stage: single high-current FET half-bridge (or full H-bridge for bidirectional) — sized for OGE hover current + 30% margin
- [ ] Dedicated switching BEC for servo rail: **6–8.4 V, ≥10 A continuous, ≥15 A peak**
- [ ] Servo BEC output isolated from FC logic supply (separate LDO or filter)
- [ ] Current sensing on main ESC output (for governor load estimation)

### RPM & Governor
- [ ] RPM sense input: tachometer pulse, **3.3 V tolerant**, filtered for noise (RC low-pass before MCU pin)
- [ ] Governor PID implemented in ESC firmware or FC (ArduPilot H_RSC_MODE = 3 for governor mode)

### Attitude Sensing
- [ ] **Dual IMU** (e.g. ICM-42688-P primary + ICM-20689 backup) — fault detection + redundancy
- [ ] IMU placed at board CoM, **soft-mounted** (foam/grommets)
- [ ] Magnetometer (e.g. IST8310 or QMC5883L) placed **away from power traces, FETs, and inductors** — minimum 20 mm clearance
- [ ] Barometer (e.g. MS5611, BMP388) for altitude + descent rate estimation (VRS prevention)

### Servo Output
- [ ] **≥4 hardware PWM timer channels** (3 CCPM + 1 tail) — all driven by the same timer peripheral for synchronisation
- [ ] PWM resolution: 16-bit timer for <1 µs jitter
- [ ] Optional: CAN or UART servo bus (FutabaS.Bus, DShot) for digital servo communication

### MCU
- [ ] STM32H7 or F7 — hardware FPU mandatory for quaternion math + CCPM mixing at 400 Hz
- [ ] ≥256 KB RAM for EKF state matrices, telemetry buffers, and PID history

### Communication
- [ ] CAN or UART telemetry for governor status, RPM, battery voltage → companion computer / GCS
- [ ] MAVLink-compatible UART for ArduPilot integration
- [ ] Dual UART: one for telemetry, one for GPS

### Safety
- [ ] Hardware watchdog (IWDG) — resets MCU if firmware hangs; servo outputs go to safe position
- [ ] Collective limit in firmware — prevent over-pitch at high headspeed (blade structural limit)
- [ ] Descent rate limit (VRS prevention): max 2.5 m/s descent in autopilot modes

---

## Sources
- [01_swash_plate.md](./01_swash_plate.md) — swash plate mechanics
- [02_rotation_orientation_control.md](./02_rotation_orientation_control.md) — PID loops, flight control architecture
- [03_drone_sensors.md](./03_drone_sensors.md) — sensor fusion, Kalman filter
- NPTEL: Drone Systems and Control, IISc Bangalore — Prof. Suresh Sundaram & Prof. Rudrashis Majumder
