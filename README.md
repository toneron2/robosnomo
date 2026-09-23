# RoboSnomo

**A 1976 Polaris Colt snowmobile made autonomous: perception, engine control and actuation
as three layers on one CAN bus.**

| | |
|---|---|
| **Status** | Architecture documented, one of eight hardware components acquired, nothing built. |
| **Repository** | Specifications only: a `CLAUDE.md` per subsystem, two research documents (vision, 1,300 lines; actuators, 1,600 lines). No code yet |
| **Budget** | about $3,981 estimated |
| **Environment** | −40 °C operation, IP66 or better |
| **Licence** | MIT |

A standalone project on this account, separate from the governance architecture.

## Architecture

| Layer | Function | Hardware | State |
|---|---|---|---|
| 1 Brain | vision, radar, sensor fusion, path planning | NVIDIA Jetson Orin Nano Super, 67 TOPS | **acquired** |
| 2 Heart | two-stroke engine control | Speeduino v0.4 (STM32F407, native CAN) | sourcing |
| 3 Muscle | steering, brake, throttle | Progressive Automations PA-04 linear actuators (400 lb; 12 in steering, 6 in brake); Dynamixel MX-106T throttle servo | sourcing / planned |

Perception: a synchronised Arducam IMX477 stereo pair and a TI AWR1843 77 GHz radar, fused
with u-blox ZED-F9P RTK GPS and an IMU in an extended Kalman filter. Detection is YOLOv8n
INT8 at about 65 FPS on the Jetson. Planning is hybrid A* globally and the dynamic window
approach locally.

```
 [Jetson Orin Nano] ──── CAN 2.0B, 500 kbit/s ──── [Speeduino ECM]
      120 Ω                                              120 Ω
   vision · radar                                     actuators
   GPS · IMU                                    steering 0x200 · brake 0x210 · throttle 0x220
```

| CAN ID range | Subsystem | Purpose |
|---|---|---|
| 0x001–0x0FF | safety | emergency stop, kill switch |
| 0x100–0x1FF | ECM | engine RPM, temperatures, throttle position |
| 0x200–0x2FF | actuators | steering, brake, throttle commands |
| 0x500 | heartbeat | alive signal every 100 ms |

## Safety

A hardware kill switch overrides all software. Each actuator carries redundant position
sensing (Hall effect and potentiometer). A CAN watchdog enters fail-safe if the heartbeat
is silent for more than 300 ms. Operation is geofenced to defined test areas, manual control
is retained on every axis, and testing proceeds in six phases from bench to field.

## Plan

| Phase | Focus | State |
|---|---|---|
| 1 | architecture and documentation | complete |
| 2 | hardware procurement | in progress |
| 3 | core systems: JetPack 6.2, CAN, TunerStudio | planned |
| 4 | vision and perception | planned |
| 5 | actuation and control | planned |
| 6 | autonomous testing | planned |

## Documents

`CLAUDE.md` at the root and in each subsystem directory carries the specification.
[`docs/vision-ai-research.md`](docs/vision-ai-research.md) and
[`docs/actuator-research.md`](docs/actuator-research.md) hold the component comparisons.

## Contact

Tony Slosar · TODOMODO.IO AGENCY LLC · anthonyslosar@gmail.com · [t.me/toneron2](https://t.me/toneron2) · [slosars.me](https://slosars.me)
