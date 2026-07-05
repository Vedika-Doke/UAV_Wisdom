# Drone Systems and Control — NPTEL

**Course:** NOC: Drone Systems and Control  
**Platform:** NPTEL  
**Institution:** IISc Bangalore  
**Instructors:** Prof. Suresh Sundaram · Prof. Rudrashis Majumder  
**Course URL:** https://nptel.ac.in/courses/101108661  

> **Study focus:** Notes are taken with an eye toward designing an **AIO (All-In-One) FC + ESC PCB for single-rotor helicopter drones**. Each lecture note includes an *AIO PCB Design Implications* section where relevant.

---

## Lecture Notes

| # | Topic | File | Status |
|---|-------|------|:------:|
| 01 | Swash-Plate Based Systems | [01_swash_plate.md](./01_swash_plate.md) | 🟢 done |
| 02 | Rotation & Orientation Control (Roll/Pitch/Yaw, IMU, PID) | [02_rotation_orientation_control.md](./02_rotation_orientation_control.md) | 🟢 done |
| 03 | Drone Sensors (MEMS Accel, Gyro, Magnetometer, Sensor Fusion) | [03_drone_sensors.md](./03_drone_sensors.md) | 🟢 done |
| — | Helicopter Reference (anatomy, control, torque, vibration, servo/ESC specs) | [helicopter.md](./helicopter.md) | 🟢 done |

---

## Key Concepts (running list)

- **Swash plate** = mechanical interface between static frame and spinning rotor; enables collective + cyclic pitch control
- **Collective pitch** → altitude control (all blades change angle together)
- **Cyclic pitch** → roll/pitch/yaw (blade angle varies sinusoidally per revolution)
- **IMU** (gyro + accel) on the AIO board detects roll/pitch/yaw → feeds PID loops
- **PID output** in a helicopter → servo positions (not motor speed); roll/pitch PID → cyclic servos; yaw PID → tail rotor; altitude PID → collective
- Helicopter AIO PCB needs: ≥4 servo channels, single high-current ESC stage, RPM sensing, vibration isolation strategy, dual IMU, MCU with hardware FPU

---

## Sources
- NPTEL course page: https://nptel.ac.in/courses/101108661
