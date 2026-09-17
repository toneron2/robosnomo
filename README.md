# Project RoboSnomo

**Converting a vintage 1976 Polaris Colt into a fully autonomous, self-driving snowmobile.**

Merging 70s mechanical engineering with modern AI technology.

**Status: Architecture documented, one of eight hardware components acquired, nothing built.**
[Hardware Status](#hardware-status) and [Timeline](#timeline) below carry the detail.

A standalone project on this account, separate from the governance architecture.

---

## Overview

Project RoboSnomo is an ambitious hobby-plus project to transform a classic 1976 Polaris Colt snowmobile into an autonomous vehicle using a Tesla-inspired 3-layer architecture:

| Layer | Name | Function | Hardware |
|-------|------|----------|----------|
| **Layer 1** | The Brain | AI, vision, path planning | NVIDIA Jetson Orin Nano Super (67 TOPS) |
| **Layer 2** | The Heart | Engine control (2-stroke ECM) | Speeduino v0.4 (STM32F407) |
| **Layer 3** | The Muscle | Steering, braking, throttle | Progressive Automations linear actuators |

**Communication**: CAN Bus 2.0B @ 500 kbps connecting all layers

---

## Features

- **Computer Vision**: Stereo cameras (IMX477) + 77GHz radar for obstacle detection
- **AI Models**: YOLOv8n-INT8 @ 65 FPS for real-time object detection
- **Sensor Fusion**: EKF fusing vision, radar, GPS (RTK), and IMU data
- **Path Planning**: Hybrid A* (global) + DWA (local) for autonomous navigation
- **Safety Systems**: Redundant sensors, hardware kill switch, CAN watchdog, fail-safe modes
- **Winter-Ready**: Designed for -40°C operation with IP66+ weatherproofing

---

## Project Structure

```
robosnomo/
├── CLAUDE.md               # Project manager documentation (detailed specs)
├── README.md               # This file
├── layer1-brain/           # AI & Vision System (Jetson Orin Nano)
├── layer2-heart/           # Engine Control Module (Speeduino ECM)
├── layer3-muscle/          # Actuation Systems (steering, brake, throttle)
├── integration/            # CAN Bus protocols & system integration
├── mechanical/             # Chassis, fabrication, mounting
├── datasets/               # AI training data & models
├── docs/                   # Technical research & datasheets
├── tools/                  # Development utilities & scripts
└── website/                # Project documentation website
```

Each subdirectory contains its own `CLAUDE.md` with detailed technical specifications.

---

## Hardware Status

| Component | Status | Notes |
|-----------|--------|-------|
| Jetson Orin Nano Super (1TB) | **Acquired** | 67 TOPS, ready for development |
| Speeduino v0.4 ECM | Sourcing | STM32F407 variant with native CAN |
| Steering Actuator | Sourcing | Progressive PA-04-12-400 (400 lbs, 12") |
| Brake Actuator | Sourcing | Progressive PA-04-6-400 (400 lbs, 6") |
| Throttle Servo | Planned | Dynamixel MX-106T (84 kg·cm) |
| Stereo Cameras | Sourcing | Arducam IMX477 synchronized kit |
| 77GHz Radar | Planned | TI AWR1843 |
| GPS (RTK) | Planned | u-blox ZED-F9P |

**Estimated Total Budget**: ~$3,981

---

## Getting Started

### Prerequisites

- NVIDIA JetPack SDK 6.2 (Ubuntu 22.04 L4T)
- Python 3.x with PyTorch, OpenCV (CUDA), TensorRT
- TunerStudio MS for Speeduino configuration
- CAN bus tools (python-can, SocketCAN)

### Development Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/robosnomo.git
   cd robosnomo
   ```

2. Read the main `CLAUDE.md` for project overview and navigation

3. Navigate to the relevant subdirectory for your work area

---

## Architecture

### CAN Bus Network

```
[Jetson Orin Nano]─────[CAN 500 kbps]─────[Speeduino ECM]
      (120Ω)                                    (120Ω)
        │                                          │
   [Vision/Radar]                           [Actuators]
        │                                    ├─ Steering (0x200)
   [GPS/IMU]                                 ├─ Brake (0x210)
                                             └─ Throttle (0x220)
```

### Message Priority

| ID Range | Subsystem | Purpose |
|----------|-----------|---------|
| 0x001-0x0FF | Safety | Emergency stop, kill switch |
| 0x100-0x1FF | ECM | Engine RPM, temps, TPS |
| 0x200-0x2FF | Actuators | Steering, brake, throttle commands |
| 0x500 | Heartbeat | System alive signal (100ms) |

---

## Safety

This is a **hobby-plus project** - safety is taken seriously:

- Hardware emergency stop (killswitch) overrides all software
- Redundant position sensors (Hall + potentiometer) on each actuator
- CAN watchdog triggers fail-safe if heartbeat timeout >300ms
- Geofencing prevents operation outside defined test areas
- Manual override capability retained on all controls
- 6-phase testing protocol from bench to field

**Never skip validation steps. Test incrementally. Document everything.**

---

## Timeline

| Phase | Focus | Status |
|-------|-------|--------|
| 1 | Architecture & Documentation | **Complete** |
| 2 | Hardware Procurement | **In Progress** |
| 3 | Core Systems Setup | Planned |
| 4 | Vision & Perception | Planned |
| 5 | Actuation & Control | Planned |
| 6 | Autonomous Testing | Planned |

---

## Contributing

Looking for contributors with expertise in:

- **Mechanics**: Vintage Polaris repair, chassis modification
- **Welding/Fabrication**: Actuator mounting, structural work
- **Software**: Python (AI), C++ (embedded), ROS 2
- **Electronics**: CAN bus, sensor integration, power systems

---

## Documentation

- `CLAUDE.md` files throughout contain comprehensive technical specifications
- `docs/vision-ai-research.md` - Detailed AI/vision research (1300+ lines)
- `docs/actuator-research.md` - Actuator comparisons and specifications (1600+ lines)

---

## License

TBD - Open source license to be determined

---

## Acknowledgments

- NVIDIA Jetson community
- Speeduino open-source ECM project
- Vintage Sleds (VS) community for Polaris expertise

---

**Project Lead**: Anthony Slosar
**Codename**: Project RoboSnomo
**Started**: 2025
