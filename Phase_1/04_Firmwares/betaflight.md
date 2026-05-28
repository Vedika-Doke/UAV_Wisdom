# Betaflight

> Fork of Cleanflight focused on FPV racing/freestyle.

## Strengths
- Lowest-latency loop, best tuned for acro
- Massive hardware support, huge community
- Configurator GUI is mature

## Limitations
- No real autonomy (no proper GPS rescue beyond beta features)
- Designed for multirotors only

## Architecture (notes to fill)
- Bare-metal, no RTOS
- 4–8 kHz gyro / 8 kHz PID loop on H7
- RPM filter (needs bidirectional DShot)

## Sources
1. https://github.com/betaflight/betaflight
