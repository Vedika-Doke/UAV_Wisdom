# ESC — Comparison Matrix

Verified against vendor pages as of May 2026. Note the **firmware column** — post-BLHeli_32 discontinuation, vendors have migrated to AM32 (32-bit) and Bluejay (8-bit).

CSV mirror: [`../matrices/escs.csv`](../matrices/escs.csv)

---

## A. 4-in-1 ESCs (the FPV / lightweight category)

| ESC | Cont. A | Burst A | Cells | MCU class | **Firmware (2026)** | Onboard BEC | Weight (g) | Mounting | Price (USD) | Notes |
|-----|--------:|--------:|-------|-----------|---------------------|-------------|-----------:|----------|------------:|-------|
| **T-Motor F55A Pro II** | 55 | 75 | 3–6S | 32-bit (F3) | **AM32** (was BLHeli_32) | 10 V / 2 A | ~14 | 30.5×30.5 | ~75 | Default 5" premium pick |
| **Hobbywing XRotor Micro 60A** | 60 | — | 3–6S | 32-bit | BLHeli_32 / migrating to AM32 | none | ~12 | 30.5×30.5 | ~70 | DShot1200, large MOSFETs |
| **Holybro Tekko32 F4 65A** | 65 | 80 | 3–6S | 32-bit (F4) | BLHeli_32 / AM32 | 9 V / 2 A | ~12 | 30.5×30.5 | ~85 | Onboard F4 for ESC telem analytics |
| Mamba F65 / F60 | 65 / 60 | — | 3–6S | 32-bit | BLHeli_32 / AM32 | yes | ~13 | 30.5 | ~85 | DiatoneRC |
| SpeedyBee BLS 50A | 50 | 60 | 3–6S | 8-bit EFM8 | **Bluejay** (BLHeli_S board) | 5 V / 2 A | ~9 | 30.5 | ~45 | **Cheap, Bluejay = bidir DShot** |
| iFlight BLITZ E55 | 55 | 65 | 3–6S | 32-bit | BLHeli_32 / AM32 | yes | ~13 | 30.5 | ~70 | |
| HGLRC Forward 60A | 60 | — | 3–6S | 32-bit | BLHeli_32 / AM32 | yes | ~12 | 30.5 | ~65 | |
| **Holybro Tekko32 F4 Mini 35A** | 35 | 45 | 2–6S | 32-bit | BLHeli_32 / AM32 | yes | ~7 | 20×20 | ~55 | **Micro 4-in-1 — Phase 2 System 2 reference** |

---

## B. AIO (FC + 4-in-1 ESC on one board) — the "System 2" reference category

| Board | FC MCU | ESC cont. A | Cells | ESC firmware | Weight (g) | Mounting | Price (USD) | Notes |
|-------|--------|------------:|-------|--------------|-----------:|----------|------------:|-------|
| **SpeedyBee F405 Mini AIO 25A** | F405 | 25 | 2–4S | **Bluejay** (BLHeli_S) | ~7 | 20×20 | ~70 | **The reference for Phase 2 System 2** |
| SpeedyBee F405 Mini AIO 40A | F405 | 40 | 3–6S | Bluejay | ~9 | 20×20 | ~95 | 6S version of above |
| HGLRC Zeus F722 AIO 60A | F722 | 60 | 3–6S | BLHeli_32 | ~11 | 25.5 | ~110 | Higher-current AIO |
| iFlight BLITZ F7 AIO | F722 | 55 | 3–6S | BLHeli_32 / AM32 | ~10 | 25.5 | ~100 | |
| BetaFPV F4 1S 5-in-1 | F411 | 5 | 1S | BLHeli_S | ~2 | 14×14 | ~35 | Tinywhoop |
| Mamba MK4 F405 AIO 25A | F405 | 25 | 2–4S | BLHeli_S → Bluejay | ~7 | 20×20 | ~65 | |

---

## C. Industrial / DroneCAN smart ESCs (the "System 1" companion category)

For a Pixhawk-class FC like Phase 2 System 1, ESCs typically live off-board and talk over CAN.

| ESC | Cont. A | Cells | Protocol | Telemetry | Notable |
|-----|--------:|-------|----------|-----------|---------|
| **Holybro Tekko32 F4 50A** (with DroneCAN add-on) | 50 | 3–6S | DShot + UART telem | full | Bridge to Pixhawk via FC |
| **mRo OUYA 30A** | 30 | 3–6S | DroneCAN | full | Industrial smart ESC |
| **CUAV NEO ESC** | 80 | 4–12S | DroneCAN | full | Heavy-lift, isolated for high voltage |
| **Zubax Myxa B** | 25 (continuous), 60 burst | 5–60 V | DroneCAN, FOC | full | Top-end industrial FOC ESC |
| T-Motor Flame 60A HV | 60 | up to 12S | PWM | basic | Heavy-lift |

For System 1 — design supports **at least DShot + UART telemetry**, ideally **DroneCAN** for new industrial ESCs.

---

## D. Decision rules of thumb

| If you're building... | Pick |
|----------------------|------|
| 5" freestyle / racing (default) | T-Motor F55A Pro II (AM32) **or** SpeedyBee BLS 50A (Bluejay) |
| Micro AIO (sub-3") | SpeedyBee F405 Mini AIO 25A or 40A |
| 7" long-range | Holybro Tekko32 F4 65A |
| Heavy-lift hex/octo, individual ESCs | T-Motor Flame series (HV) |
| Industrial Pixhawk drone | Zubax Myxa or mRo OUYA via DroneCAN |

---

## E. Sourcing notes (for India / Phase 2 procurement)

- **Best Indian resellers (2026):** RaceDayQuads-IN, Quadkart, RobotShop India, Robu.in.
- Lead times: 2–6 weeks for premium gear (T-Motor, Holybro) from US suppliers. Cheaper SpeedyBee parts are usually stocked locally.
- Import duty: ~30 % all-in on drone components; budget accordingly.

---

## Sources
1. T-Motor F55A Pro II — https://store.tmotor.com/product/f55a-pro-v2-4in1-fpv-esc.html
2. Hobbywing XRotor Micro 60A — https://www.getfpv.com/hobbywing-xrotor-micro-60a-6s-4-in-1-blheli32-esc.html
3. Holybro Tekko32 F4 — https://holybro.com/collections/tekko-esc
4. SpeedyBee BLS / AIO — https://www.speedybee.com/
5. iFlight BLITZ E55 / F7 AIO — https://shop.iflight.com/
6. Zubax Myxa — https://zubax.com/products/myxa
7. CUAV NEO ESC — https://store.cuav.net/
