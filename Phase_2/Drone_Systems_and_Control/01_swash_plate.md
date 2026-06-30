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

**Think of it as:** turning a volume knob for lift — every blade tilts by the same amount at the same time.

The swash plate slides **up or down** along the rotor shaft without tilting. All pitch links move equally, so every blade bites into the air more (or less) aggressively at once. More pitch → more lift → helicopter rises.

![Collective pitch animation](https://upload.wikimedia.org/wikipedia/commons/c/c7/Helicopter_Collective.gif)
*Fig 1 — Collective input: swash plate rises/falls flat, all blades change pitch equally → altitude control.*
*(Source: Wikimedia Commons, CC-BY-SA)*

| Swash plate motion | Effect on blades | Flight result |
|--------------------|-----------------|---------------|
| Slides up (flat) | All blades increase pitch | More lift → climb |
| Slides down (flat) | All blades decrease pitch | Less lift → descend |

---

### Cyclic Pitch Control

**Think of it as:** making the rotor disc nod or tilt in a chosen direction to push the helicopter that way.

The swash plate **tilts** (instead of sliding). Because the rotating plate is linked to it, each blade's pitch changes sinusoidally — it reaches maximum pitch on one side of the disc and minimum pitch on the opposite side. The side with higher pitch generates more lift, which tilts the whole rotor disc toward the lower-lift side, vectoring the thrust.

![Cyclic pitch animation](https://upload.wikimedia.org/wikipedia/commons/4/44/Helicopter_Cyclic.gif)
*Fig 2 — Cyclic input: swash plate tilts, each blade's pitch varies once per revolution → disc tilts → helicopter translates.*
*(Source: Wikimedia Commons, CC-BY-SA)*

| Swash plate motion | Max pitch at | Disc tilts toward | Flight result |
|--------------------|-------------|-------------------|---------------|
| Tilt forward | Rear of disc | Forward | Helicopter pitches/moves forward |
| Tilt right | Left of disc | Right | Helicopter rolls/moves right |

> **Why does more pitch on one side tilt the disc toward the *other* side?**  
> Due to **gyroscopic precession** — a force applied to a spinning disc acts 90° later in the direction of rotation. So maximum pitch is commanded 90° ahead of where you want the disc to tilt.

---

### Swash Plate in Action — Combined View

![Swash plate neutral](https://upload.wikimedia.org/wikipedia/commons/5/54/HelicopterSwashPlate_Flat.gif)
*Fig 3 — Neutral swash plate: rotating and fixed plates spin/stay together, blades at constant pitch → steady hover.*
*(Source: Wikimedia Commons, CC-BY-SA)*

![Swash plate tilted](https://upload.wikimedia.org/wikipedia/commons/6/68/HelicopterSwashPlate_Tilted.gif)
*Fig 4 — Tilted swash plate (cyclic): fixed plate tilts, rotating plate follows, blade pitch varies once per revolution.*
*(Source: Wikimedia Commons, CC-BY-SA)*

![Swash plate raised](https://upload.wikimedia.org/wikipedia/commons/0/05/HelicopterSwashPlate_Raised.gif)
*Fig 5 — Raised swash plate (collective): both plates slide up, all blades increase pitch simultaneously.*
*(Source: Wikimedia Commons, CC-BY-SA)*

Together, collective and cyclic give the pilot independent control over **how much total thrust** the rotor produces and **which direction** that thrust is pointed — enabling hover, climb, descent, and translation with a single rotor.

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
- Wikipedia: [Swashplate (aeronautics)](https://en.wikipedia.org/wiki/Swashplate_(aeronautics))
- Wikipedia: [Helicopter flight controls](https://en.wikipedia.org/wiki/Helicopter_flight_controls)
- Animations by Richard Wheeler (Zephyris), Wikimedia Commons, CC-BY-SA
