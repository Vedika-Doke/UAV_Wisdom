# Applications of Rotation in Drone Operations

> Source: NPTEL — Drone Systems and Control (IISc)

---

## Rotation (Orientation Control)

Refers to changing the drone's orientation in space — described by three angles:

| Axis | Rotation | What it looks like |
|------|----------|--------------------|
| X (longitudinal) | **Roll** | Tilting left/right |
| Y (lateral) | **Pitch** | Nosing up/down |
| Z (vertical) | **Yaw** | Spinning left/right on the spot |

---

## Why It Matters

- Face a camera or sensor in the correct direction (payload pointing)
- Stabilise mid-air — especially in wind (disturbance rejection)

---

## Reference Frames

| Frame | Fixed to | Used for |
|-------|----------|----------|
| **Inertial (World)** | Ground — does not move with the drone | GPS, navigation, mapping |
| **Body (Local)** | Drone centre of mass — moves with it | Force, torque, control calculations |

Two common inertial frame conventions used in UAVs:

### ENU — East-North-Up
- X → East, Y → North, Z → Up
- Tangent-plane, centered at a reference point on Earth's surface; does not rotate with UAV
- Used in: GPS navigation, geographic mapping, GIS, ROS (default)

### NED — North-East-Down
- X → North, Y → East, Z → Down (altitude *decreases* as Z increases — aviation convention)
- Moves with the UAV but stays aligned to Earth's NED directions
- Used in: UAV autopilots, flight control systems, ArduPilot/PX4 (default)
- **Preferred in aerospace** because Z-down matches how altimeters and barometers work

| | ENU | NED |
|-|-----|-----|
| Z-axis | Up | Down |
| Default in | ROS, GIS | ArduPilot, PX4, aerospace |
| Applications | Mapping, GPS data | Flight control, stabilisation, path planning |

**GPS works in ENU/inertial frame; IMU measures in the body frame (NED convention).** The FC converts between them using the rotation matrix / quaternion every control cycle.

---

## How It Is Used

### 1. IMU — Inertial Measurement Unit
- Detects current orientation (roll, pitch, yaw) using a **gyroscope** (angular rate) and **accelerometer** (gravity vector)
- Typically a MEMS chip (e.g. MPU-6000, ICM-42688) soldered directly onto the AIO FC board
- Runs at 1–8 kHz to keep up with fast dynamics

### 2. Rotation Matrices / Quaternions
- **Rotation matrix** — 3×3 matrix that converts a vector from the **body frame** (relative to the drone) to the **world frame** (relative to the ground), or vice versa
- **Quaternion** — a 4-number representation preferred in firmware because it avoids **gimbal lock** (a singularity that occurs with Euler angles when pitch = ±90°)
- The FC firmware maintains the current attitude as a quaternion, updated every IMU sample

### 3. PID Controller — Real-time Orientation Correction

```
Error = Target orientation − Current orientation (from IMU)
PID output = Kp·error + Ki·∫error dt + Kd·d(error)/dt
```

The PID output drives the actuators to correct the error.

---

## Is This Used in Helicopter Drones Like Ours?

**Yes — this is the core of the AIO FC board's job.**

| Concept | Multirotor | Helicopter (our target) |
|---------|-----------|------------------------|
| IMU measures roll/pitch/yaw | Yes | Yes — same chip, same role |
| PID corrects orientation | Yes | Yes — runs every IMU cycle |
| PID output goes to… | Motor speed (ESC) | **Servo positions** (swash plate) |
| Rotation matrices / quaternions | Used in firmware | Used in firmware — identical math |

In a helicopter, the PID loops are:
- **Roll PID** → lateral cyclic servo command
- **Pitch PID** → longitudinal cyclic servo command
- **Yaw PID** → tail rotor servo/motor command
- **Altitude PID** → collective servo command

The swash plate translates these servo commands into actual blade pitch changes (see [01_swash_plate.md](./01_swash_plate.md)).

---

## Real-World Task Applications

How rotation (orientation) and translation (position) work together across common drone tasks:

| Task | Role of Rotation / Translation | Brief explanation |
|------|-------------------------------|-------------------|
| **Autonomous navigation** | Compute position & orientation to move | The FC must know *where it is* (translation, from GPS/barometer) and *which way it's pointing* (rotation, from IMU) simultaneously. Both are fused to plan and execute a move to a target waypoint. |
| **Target tracking** | Align camera (rotate) and follow (translate) | First yaw/pitch to point the camera at the target (rotation), then command the helicopter to close the distance (translation). These run in parallel in a tracking loop. |
| **Landing on a platform** | Adjust orientation to level, translate to center | Level the airframe (roll/pitch → 0°) while simultaneously drifting sideways over the pad centre. Precision landing uses both attitude and position control at the same time. |
| **Aerial photography** | Rotate camera gimbal, stabilise with roll/pitch | The gimbal actively counter-rotates against the drone's roll/pitch/yaw to keep the camera horizon-level. The FC provides attitude data to the gimbal controller in real time. |
| **Formation flying** | Maintain orientation & relative distance | Each drone in the formation must hold its own attitude (rotation) while also keeping a fixed offset from its neighbours (translation). Requires inter-drone communication + tight PID loops on both. |

> For a helicopter specifically, **every single task above depends on the swash plate servo loop** responding correctly to PID commands — reinforcing why servo latency and IMU accuracy on the AIO PCB are non-negotiable.

---

## AIO PCB Design Implications

- **IMU placement is critical** — mount as close to the centre of mass as possible, isolated from motor/servo vibration. Use foam or rubber grommets between IMU and PCB if possible.
- **IMU orientation matters** — the firmware needs to know which axis on the chip maps to which axis of the airframe. Document the chip orientation on the PCB silkscreen.
- **Two IMUs** (primary + backup, e.g. ICM-42688 + MPU-6000) is best practice for flight-critical boards — gives fault detection and redundancy.
- PID computation runs on the **main MCU** (e.g. STM32F4/H7) — ensure the MCU has hardware FPU (floating-point unit) since quaternion math is float-heavy.
- Servo PWM outputs must be **low-latency** — use hardware timers, not bit-banged GPIO.

---

## Sources
- NPTEL: Drone Systems and Control, IISc Bangalore — Prof. Suresh Sundaram, Prof. Rudrashis Majumder
- Related: [01_swash_plate.md](./01_swash_plate.md) — how PID outputs map to swash plate servo commands
