# UAV Types — Comparison

A UAV (Unmanned Aerial Vehicle) is any powered aircraft without a human onboard. "Drone" is the colloquial term. The interesting axis is **how they generate lift**, because everything else — endurance, payload, hovering ability, regulatory class, cost — falls out of that one choice.

---

## 1. The three architectural families

### Multirotor
Lift comes entirely from rotor thrust. Most commonly 4 motors (quadcopter), sometimes 6 (hex) or 8 (octo) for payload or redundancy.

- ✅ Hover, vertical takeoff/landing (VTOL), trivial control, mechanically simple
- ❌ Energy-inefficient — rotors are *always* fighting gravity. Battery flight time is typically **20–45 min**.
- Speed cap: ~150 km/h (most are far slower).

### Fixed-wing
Lift comes from a wing moving through air. Needs forward velocity to fly.

- ✅ Very efficient — wings generate lift "for free" once moving. Flight times of **1–24+ hours** depending on size.
- ✅ High speed and range. Large payload-to-power ratio.
- ❌ Cannot hover. Needs runway / launcher / catapult to take off, runway or net or belly-landing to land.

### VTOL (hybrid)
Vertical takeoff/landing **plus** efficient cruise. Two common architectures:
- **Tilt-rotor / tilt-wing:** motors rotate from vertical (hover) to horizontal (cruise). E.g. Wingtra, V-22 Osprey.
- **Quad-plane / "four-plus-one":** dedicated vertical lift motors *plus* a horizontal pusher/puller for cruise. Simpler mechanically, slightly less efficient.

- ✅ Best of both — no runway, but cruise-efficient
- ❌ Heavier, more complex, more failure modes, more expensive

### Single-rotor (helicopter)
One large main rotor + tail rotor for yaw. Mostly industrial / agricultural (spraying drones from XAG, DJI Agras) or large military.

- ✅ Very high payload-to-power ratio for hovering vehicles
- ❌ Complex mechanics (swashplate), dangerous to be near, expensive

---

## 2. Side-by-side

| Type | Endurance | Cruise speed | Payload | Hover | Takeoff/land | Mechanical complexity | Cost |
|------|-----------|--------------|---------|:-----:|--------------|----------------------|------|
| Multirotor | 20–45 min | 30–80 km/h | Low–Med (g–kg) | ✅ | Anywhere | Low | $ |
| Fixed-wing | 1–24+ hr | 60–200 km/h | Med–High | ❌ | Runway/launch/net | Low–Med | $$ |
| VTOL | 1–8 hr | 60–120 km/h | Med | ✅ | Anywhere | High | $$$ |
| Helicopter | 30–120 min | 60–150 km/h | High | ✅ | Anywhere | Very high | $$$$ |

---

## 3. Sub-categories from the project brief (mapped to architectures)

| Brief category | Typical architecture | Why |
|----------------|---------------------|-----|
| **Commercial photography drones** | Multirotor | Need to hover, frame shots, fly slowly. Examples: DJI Mavic 4 Pro, DJI Air 3S, Autel EVO Lite+. |
| **Surveillance drones** | Multirotor (close-range) or fixed-wing/VTOL (long-range) | Hover for static observation vs. loiter for area coverage. Examples: Skydio X10, Parrot ANAFI USA, Quantum Systems Trinity F90+. |
| **Load-carrying drones** | Multirotor (hex/octo) or helicopter | Multirotor wins for stable hover under load. Examples: DJI FlyCart 30 (30 kg payload), Freefly Alta X (15 kg). |
| **Long-range UAVs** | Fixed-wing or VTOL | Energy efficiency dominates. Examples: Wingtra WingtraRAY (mapping), JOUAV CW-15 (VTOL, 3 hr / 200 km). |

---

## 4. Regulatory shorthand (India / global)

- **Nano / micro:** < 250 g — usually permission-free (still no-fly zones apply)
- **Small:** 250 g – 2 kg — registration required
- **Medium:** 2 – 25 kg — permits required, pilot certification
- **Large:** > 25 kg — heavily regulated, often equivalent to manned aircraft rules

In India, all UAVs must be registered with **DGCA** on the **Digital Sky** platform. The unique identifier scheme is **UIN**. Operator certification is **RPC** (Remote Pilot Certificate).

Outside India: FAA Part 107 (USA), EASA Open/Specific/Certified (EU), CAA (UK).

---

## 5. Reference vehicles to study

Pick one from each category and read its datasheet end-to-end — it accelerates your understanding of what's normal for that class.

| Class | Recommended reference |
|-------|----------------------|
| FPV freestyle 5" | iFlight Nazgul Evoque F5D (or any 5" build on Oscar Liang) |
| Camera multirotor | DJI Mavic 3 Pro (close-source but well-documented) |
| Industrial multirotor | Freefly Alta X — open hardware, PX4-compatible |
| Fixed-wing | Quantum Systems Trinity F90+ (mature commercial) |
| VTOL | Wingtra WingtraRAY (tilt-rotor, NDAA-compliant) |
| Long-range FPV | iFlight Chimera 7" / 9" |

---

## 6. What this means for Phase 2

The brief asks us to design FCs for **photography drones, surveillance drones, and load-carrying drones** — all **multirotor**. So Phase 2 design effort is concentrated on:

- Multirotor-compatible FC firmware (PX4 / ArduPilot Copter)
- Hover-stable EKF tuning
- Payload mounting / vibration isolation
- Power architecture sized for hex/octo current draws

We do *not* need to support fixed-wing aerodynamic surfaces (servos for ailerons/elevators), simplifying the FC's I/O.

---

## Sources

1. Mugin UAV — *Fixed-Wing vs Multirotor UAVs* — https://www.muginuav.com/fixed-wing-vs-multirotor-uavs-industrial-use-mugin/
2. Grepow — *Multirotor vs Fixed Wing vs VTOL* — https://www.grepow.com/blog/multirotor-vs-fixed-wing-vs-vtol-uav-what-is-the-difference.html
3. AUAV — *Drone Types* — https://www.auav.co.uk/articles/drone-types/
4. Wingtra — *WingtraRAY VTOL drone* — https://wingtra.com/vtol-drone/
5. JOUAV — *VTOL Drone* — https://www.jouav.com/vtol-drone
6. DGCA Digital Sky (India) — https://digitalsky.dgca.gov.in/
7. UAV Coach — *DJI Alternatives 2026* (NDAA context) — https://uavcoach.com/dji-alternatives/
