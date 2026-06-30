# Swash-Plate Based Systems

> Source: NPTEL — Drone Systems and Control (IISc), Slide 11

---

## What is a Swash Plate?

A **mechanical interface** between the stationary frame (body of the UAV) and the rotating rotor blades.

It consists of two linked plates:

| Plate | Behaviour |
|-------|-----------|
| **Fixed Plate** | Connected to control inputs; does **not** rotate |
| **Rotating Plate** | Mounted on the rotor shaft; rotates with the blades |

Control inputs applied to the fixed plate are transferred through the linkage to the rotating plate, which then changes individual blade pitch angles as the rotor spins.

---

## Purpose in UAV Systems

### Collective Pitch Control
- Uniformly changes the pitch angle of **all blades simultaneously**
- Primary mechanism for **altitude control** (more collective → more lift)

### Cyclic Pitch Control
- Varies the pitch angle of each blade **as a function of its angular position** during rotation
- Controls **roll, pitch, and yaw** — one complete revolution produces a sinusoidal pitch variation
- Tilts the rotor disc to vector thrust in the desired direction

Together these two controls give helicopters the ability to hover, translate, and manoeuvre with a single rotor.

---

## Advantages (vs. fixed-pitch multirotors)

| Advantage | Detail |
|-----------|--------|
| Dynamic Maneuverability | Precise control of flight orientation and stability |
| Efficient Hovering | Suitable for stationary-flight tasks (surveillance, mapping) |
| Lift Capacity | High thrust-to-weight ratio — better for heavier payloads |
| VTOL | No runway required for takeoff or landing |

---

## Challenges

| Challenge | Impact |
|-----------|--------|
| Mechanical Complexity | More parts → harder to design, maintain, and miniaturise onto a PCB-integrated platform |
| Weight | Adds mass compared to fixed-pitch multirotor designs |
| Cost | Higher manufacturing and operational cost |
| Vibration Sensitivity | Mechanical wear and vibration degrade sensors and solder joints — critical consideration for AIO PCB placement and damping |

---

## AIO PCB Design Implications

> Relevance to the helicopter AIO FC+ESC design goal:

- **Vibration** from the swash plate linkage is a primary enemy of surface-mount components; the PCB must include vibration isolation mounts or soft-mounting strategy.
- Collective + cyclic demands **three independent servo channels** minimum (two cyclic, one collective) — the AIO board must expose at least 3 PWM/digital servo outputs.
- Tail rotor yaw control adds a 4th channel.
- The ESC section must handle a **single large brushless motor** (main rotor) rather than 4 smaller motors — different current profile: high sustained current, lower peak switching frequency.
- Rotor RPM sensing (for governor mode) should be integrated — tachometer input or back-EMF sensing pin on the ESC stage.

---

## Sources
- NPTEL: Drone Systems and Control, IISc Bangalore — Prof. Suresh Sundaram, Prof. Rudrashis Majumder (Slide 11)
