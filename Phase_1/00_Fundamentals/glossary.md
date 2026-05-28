# Glossary

Acronyms and protocols that show up across docs, forums, and product pages. Grouped by where they appear in the stack.

---

## Vehicle / airframe
| Term | Stands for / meaning |
|------|----------------------|
| UAV | Unmanned Aerial Vehicle — the aircraft |
| UAS | Unmanned Aircraft System — UAV + ground equipment + comms |
| RPV / RPAS | Remotely Piloted (Aerial) System — formal term used by regulators |
| VTOL | Vertical Take-Off and Landing |
| MTOW | Maximum Take-Off Weight |
| AUW | All-Up Weight (current actual weight including battery + payload) |
| TWR | Thrust-to-Weight Ratio — >2 to fly, >4 for agility |
| KV | Motor speed constant — RPM per volt with no load |

## Flight controller / firmware
| Term | Stands for / meaning |
|------|----------------------|
| FC | Flight Controller |
| FMU | Flight Management Unit (Pixhawk's term for an FC) |
| MCU | Microcontroller Unit — the FC's brain (STM32 F4/F7/H7) |
| RTOS | Real-Time Operating System — NuttX (PX4), ChibiOS (ArduPilot) |
| HAL | Hardware Abstraction Layer |
| EKF | Extended Kalman Filter — state estimator fusing IMU/GPS/baro |
| AHRS | Attitude & Heading Reference System |
| PID | Proportional-Integral-Derivative controller |
| BBL | Black Box Log — recorded flight data for post-flight analysis |

## Sensors
| Term | Meaning |
|------|---------|
| IMU | Inertial Measurement Unit — accel + gyro (sometimes + mag) |
| Accel | Accelerometer — measures linear acceleration incl. gravity |
| Gyro | Gyroscope — measures angular rate (°/s) |
| Mag | Magnetometer — measures Earth's magnetic field for heading |
| Baro | Barometer — air pressure for altitude |
| TOF | Time-of-Flight rangefinder — short-range altitude / obstacle |
| RTK | Real-Time Kinematic — centimetre-grade GPS using base station |
| PPK | Post-Processed Kinematic — same idea, processed after flight |

## Motors / ESC
| Term | Meaning |
|------|---------|
| BLDC | Brushless DC motor |
| ESC | Electronic Speed Controller |
| BEC | Battery Eliminator Circuit — buck regulator giving 5 V / 3.3 V |
| PWM | Pulse Width Modulation — legacy 1–2 ms motor signal |
| OneShot125 / OneShot42 | Faster analog motor protocols (now legacy) |
| Multishot | Even faster analog protocol (now legacy) |
| DShot | Digital ESC protocol — DShot150 / 300 / 600 / 1200 |
| Bidirectional DShot | DShot + ESC telemetry back to FC each frame (enables RPM filter) |
| KISS | KISS ESC telemetry protocol (separate UART) |
| BLHeli_S / _32 / AM32 / Bluejay | ESC firmwares (see ESC notes) |
| FOC | Field-Oriented Control — advanced sinusoidal motor drive |

## Radio link (RC)
| Term | Meaning |
|------|---------|
| RC | Radio Control |
| Tx | Transmitter (the pilot's handheld) |
| Rx | Receiver (on the drone) |
| PWM | Per-channel pulse-width — one wire per channel (legacy) |
| PPM | Pulse-Position Modulation — all channels on one wire (legacy) |
| SBUS | Serial protocol from Futaba — inverted UART, 16 channels |
| IBUS | FlySky's serial protocol |
| CRSF | Crossfire Serial — TBS protocol used by Crossfire and ELRS |
| ELRS | ExpressLRS — open-source long-range RC link |
| LBT / FCC | Listen-Before-Talk vs FCC RF regulatory modes |

## Video link (FPV)
| Term | Meaning |
|------|---------|
| VTX | Video Transmitter (5.8 GHz analog or digital) |
| VRX | Video Receiver (in goggles / ground station) |
| OSD | On-Screen Display — telemetry overlay |
| MSP DisplayPort | Protocol Betaflight uses to drive digital VTX OSDs |
| HDZero / Walksnail / DJI O3/O4 | Digital FPV systems |

## Comms / autonomy
| Term | Meaning |
|------|---------|
| GCS | Ground Control Station — QGroundControl, Mission Planner |
| MAVLink | Messaging protocol between FC ↔ GCS ↔ companion computer |
| MAVSDK | High-level SDK on top of MAVLink (C++/Python/Swift) |
| MSP | MultiWii Serial Protocol — used by Betaflight/INAV configurators |
| DroneCAN | CAN-bus protocol for smart peripherals (formerly UAVCAN v0) |
| CAN-FD | CAN with Flexible Data-rate — faster than classic CAN |
| micro-ROS / uXRCE-DDS | ROS 2 talking directly to a microcontroller (PX4 v1.14+) |

## Buses / electrical
| Term | Meaning |
|------|---------|
| UART | Universal Async Receiver/Transmitter — serial port |
| I²C | Inter-Integrated Circuit — 2-wire bus (mag, baro, OLED) |
| SPI | Serial Peripheral Interface — high-speed (IMU, flash) |
| CAN | Controller Area Network — robust differential bus |
| LiPo | Lithium-Polymer battery |
| Li-Ion | Lithium-Ion (18650 / 21700 cells) — higher energy density, lower C |
| S / P | Cells in Series / Parallel (e.g. "4S2P" = 4 series × 2 parallel) |
| C-rating | Continuous discharge rate (amps = C × Ah) |

## Regulatory
| Term | Meaning |
|------|---------|
| DGCA | Directorate General of Civil Aviation (India) |
| Digital Sky | DGCA's UAS registration / flight authorisation portal |
| UIN | Unique Identification Number (DGCA registration) |
| RPC | Remote Pilot Certificate (India) |
| Part 107 | FAA's commercial drone rules (USA) |
| NDAA-compliant | Built without certain Chinese components — required for US gov work |
| Blue UAS | DoD-approved drone list (USA) |
| Remote ID | Broadcast identification beacon required for many drones globally |

> *Add new terms as they show up while reading.*
