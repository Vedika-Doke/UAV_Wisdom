# Flight Controller Internals & Peripherals

What's inside an FC, and what hangs off it.

## Inside the FC
| Block | Function | Example parts |
|-------|----------|---------------|
| MCU | Runs the firmware loop | STM32F405 / F745 / H743 / H753 |
| IMU | Accel + gyro | ICM-42688-P, BMI270, ICM-20689 |
| Barometer | Altitude (pressure) | BMP388, DPS310, MS5611 |
| Magnetometer | Heading | IST8310, RM3100 |
| Power module | Voltage/current sensing | onboard or external |
| OSD chip | Video overlay | AT7456E (analog), MAX7456 |
| Blackbox flash | Log storage | onboard SPI flash or SD |
| Comm buses | UART · I²C · SPI · CAN | — |

## Outside the FC (peripherals)
- GPS module (often with mag + safety switch)
- RF / telemetry radios (SiK 433/915 MHz, ELRS backpack)
- External magnetometer (away from power wiring)
- Companion computer (Raspberry Pi, Jetson) over UART/Ethernet
- Smart battery (over SMBus / DroneCAN)

## Sensor redundancy (commercial grade)
- Triple-redundant IMU (different vendors → different failure modes)
- Dual barometer
- Isolated power rails for sensors vs. servos
- Watchdog + brownout protection

## Sources
<!-- 1. -->
