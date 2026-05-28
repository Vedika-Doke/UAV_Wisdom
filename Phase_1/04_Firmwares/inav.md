# INAV

> *"International Navigation"* — fork of Cleanflight/Betaflight (2015), aimed at GPS-based **navigation** for multirotors *and* fixed-wing aircraft.

INAV is the middle ground between Betaflight (no autonomy) and ArduPilot (everything-and-the-kitchen-sink). If you're flying a long-range cruiser, a wing, or a beginner mission-capable drone, this is often the sweet spot.

---

## 1. Philosophy

- **Multi-vehicle from day one** — multirotor, fixed-wing, VTOL all in the same firmware (unlike Betaflight).
- **GPS-first** — assumes you have a GPS module; waypoint missions are first-class.
- **Approachable** — keeps Betaflight's configurator paradigm, so the learning curve is short for FPV pilots upgrading.
- **Failsafe-obsessed** — designed assuming the link will drop and the drone has to handle it.

---

## 2. What it adds over Betaflight

| Feature | INAV | Betaflight |
|---------|:---:|:---:|
| Waypoint missions | ✅ | ❌ |
| RTH (Return To Home) | ✅ full | partial (GPS Rescue) |
| Loiter / position hold | ✅ | ❌ |
| Altitude hold | ✅ | ❌ |
| Course-correction navigation | ✅ | ❌ |
| Fixed-wing support | ✅ | ❌ |
| VTOL (quad-plane) | ✅ | ❌ |
| Mixer for unusual configurations | ✅ | limited |
| Configurator | INAV Configurator | Betaflight Configurator |

---

## 3. What it doesn't try to do (vs ArduPilot/PX4)

- No **EKF2/EKF3** estimator (uses a simpler complementary fusion + GPS).
- Smaller vehicle library (no rover, sub, blimp).
- Less mature **MAVLink** integration — you can configure missions via Mission Planner but it's not as deep.
- No companion-computer ecosystem (no native ROS, no MAVSDK).
- Closed user community vs ArduPilot/PX4's research scale.

In return, you get a much **simpler setup** — a 2-hour novice can get a wing flying a mission with INAV. ArduPilot takes longer.

---

## 4. Architecture

- Bare-metal, like Betaflight (no RTOS).
- Same general task-priority scheduler.
- Adds a **navigation subsystem** running at ~10 Hz to compute mission waypoints, ETA, and high-level setpoints, which feed into the same inner attitude/rate loops.

---

## 5. Hardware support

- **MCUs:** F405, F411, F722, F745, H743 (same Betaflight target families).
- **Specifically:** Matek's F405-WING, F722-WING, H743-WING families are the *de facto* INAV reference boards — Matek and INAV co-evolve.
- IMUs: ICM-42688P, ICM-20689, MPU6000 — same as Betaflight.

---

## 6. Configurator

The **INAV Configurator** is forked from Betaflight Configurator. Same Chrome-app feel; adds tabs for:
- Mission Control (planning waypoints on a map)
- Failsafe / RTH config
- Navigation tuning (P/I/D for position hold)
- Mixer (custom motor/servo allocation for unusual airframes)

There's also a separate **INAV Blackbox Explorer** for log analysis.

---

## 7. When to use INAV

| If the drone is... | Use INAV? |
|--------------------|:---------:|
| Fixed-wing for FPV or mapping | ✅ Default |
| Long-range multirotor cruiser | ✅ |
| VTOL quad-plane (hobby/research) | ✅ |
| Wants RTH + loiter without full autonomy stack | ✅ |
| Pure FPV freestyle | ❌ → Betaflight |
| Commercial mapping / inspection drone | ⚠️ ArduPilot is more mature |
| Research / multi-agent / ROS integration | ❌ → PX4 |

---

## 8. For Phase 2

INAV is **not the primary target** for either Phase 2 system, but worth noting:
- If the **System 2 AIO** could also support INAV (same hardware, alternate build target), that's a marketing plus — users can pick FPV-mode (Betaflight) or autonomy-mode (INAV) on the same board.
- INAV targets are easier to upstream than ArduPilot targets, which makes Phase 2 → community integration cheap.

## Sources
1. INAV GitHub — https://github.com/iNavFlight/inav
2. INAV Wiki — https://github.com/iNavFlight/inav/wiki
3. INAV Configurator — https://github.com/iNavFlight/inav-configurator
4. Painless360 — *INAV setup tutorials* — YouTube
5. Matek Systems product line — https://www.mateksys.com/
