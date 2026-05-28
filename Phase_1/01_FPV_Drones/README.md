# 01 — FPV Drones

FPV (First-Person View) drones are the learning platform for Phase 1 — small, modular, well-documented, and they expose every layer of the stack.

## Contents
- [frames-motors-props.md](./frames-motors-props.md)
- [radio-vtx-receivers.md](./radio-vtx-receivers.md)
- [batteries-power.md](./batteries-power.md)
- [example-builds.md](./example-builds.md)

## Anatomy of an FPV quad

```mermaid
flowchart LR
    RC[RC Transmitter] -- 2.4GHz --> RX[Receiver]
    RX -- SBUS/CRSF --> FC[Flight Controller]
    FC -- DShot --> ESC
    ESC --> M1[Motor 1]
    ESC --> M2[Motor 2]
    ESC --> M3[Motor 3]
    ESC --> M4[Motor 4]
    BAT[LiPo Battery] --> PDB[PDB / AIO]
    PDB --> ESC
    PDB --> FC
    CAM[FPV Camera] --> VTX
    VTX -- 5.8GHz --> GOGGLES[Goggles]
    FC -.OSD.-> VTX
```

## Status
🟡 in progress
