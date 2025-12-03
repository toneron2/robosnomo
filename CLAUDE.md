# Project RoboSnomo - Project Management & Orchestration
## Autonomous 1976 Polaris Colt Conversion

**Role**: This CLAUDE.md file serves as the **Project Manager (PM) level** - for high-level coordination, cross-cutting concerns, and navigation between sub-projects.

**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

## Project Overview

**Mission**: Convert a vintage 1976 Polaris Colt into a fully autonomous, self-driving vehicle, merging 70s mechanical engineering with modern AI technology.

**Codename**: Project RoboSnomo
**Architecture**: Tesla-inspired 3-layer autonomous vehicle system at hobbyist-plus level
**Budget**: ~$3,981 for complete system
**Timeline**: Winter 2025 → Fall 2025 (hardware → field testing)

---

## Project Directory Structure & Navigation

This project is organized into **9 specialized subdirectories**, each with its own focused CLAUDE.md containing domain-specific technical knowledge.

```
robosnomo/                          # PROJECT ROOT (PM Level - you are here)
├── CLAUDE.md                       # This file - PM orchestration & navigation
│
├── website/                        # Public-facing documentation
│   ├── index.html                  # Comprehensive technical website (Cloudflare ready)
│   └── snowmobile-bg.mp4           # Website background video
│
├── layer1-brain/                   # AI & Vision System (NVIDIA Jetson)
│   └── CLAUDE.md                   # Jetson specs, vision pipeline, sensor fusion, path planning
│
├── layer2-heart/                   # Engine Control Module (Speeduino ECM)
│   └── CLAUDE.md                   # ECM specs, 2-stroke config, fuel/ignition mapping, TunerStudio
│
├── layer3-muscle/                  # Actuation Systems (Steering, Brake, Throttle)
│   └── CLAUDE.md                   # Actuator specs, fail-safes, power requirements, linkage design
│
├── integration/                    # CAN Bus & System Integration
│   └── CLAUDE.md                   # CAN protocol, message IDs, wiring, error handling, testing
│
├── mechanical/                     # Chassis, Fabrication, Mounting
│   └── CLAUDE.md                   # Polaris Colt restoration, welding, brackets, weatherproofing
│
├── datasets/                       # Training Data & AI Models
│   └── CLAUDE.md                   # Winter datasets, YOLOv8 training, TensorRT optimization
│
├── docs/                           # Technical Documentation & Research
│   ├── CLAUDE.md                   # Reference materials repository
│   ├── vision-ai-research.md       # Comprehensive vision/AI research
│   └── actuator-research.md        # Actuator technical comparisons
│
└── tools/                          # Development Utilities & Scripts
    └── CLAUDE.md                   # CAN utils, calibration tools, benchmarks, testing scripts
```

---

## Entry/Exit Process - How to Navigate Sub-Projects

### Working on a Sub-Project

**Entry Process**:
1. **Identify the subsystem** you need to work on from the directory structure above
2. **Navigate to that subdirectory** (e.g., `layer1-brain/` for vision work)
3. **Read the subdirectory's CLAUDE.md** to get domain-specific technical context
4. **Work within that directory** - all files, code, and documentation for that subsystem live there
5. **Reference the integration/ directory** for cross-cutting CAN bus protocol details if needed

**Example**:
- Working on stereo camera calibration? → `layer1-brain/`
- Tuning Speeduino ECM for 2-stroke? → `layer2-heart/`
- Designing steering actuator mounts? → `mechanical/` (reference `layer3-muscle/` for specs)
- Implementing CAN message protocol? → `integration/`

**Exit Process**:
1. **Document your work** in the appropriate subdirectory (code, notes, test results)
2. **Update the subdirectory's CLAUDE.md** if you made significant architectural changes
3. **Return to this root CLAUDE.md** (PM level) when you need to:
   - Coordinate between multiple subsystems
   - Update the centralized BOM
   - Check timeline/milestones
   - Prepare for GitHub commits
   - Work on cross-cutting concerns (power budget, safety, integration testing)

### Working at PM Level (Root Directory)

**When to stay at PM level**:
- Coordinating integration between multiple layers (e.g., CAN bus message changes affecting Brain + Muscle)
- Updating centralized BOM or budget tracking
- Managing project timeline and milestones
- Preparing GitHub repository (initialization, branching, releases)
- Reviewing cross-cutting concerns (power budget, safety systems, testing coordination)
- Deploying/updating the public website

**When to navigate to a subdirectory**:
- Writing code for a specific subsystem
- Researching technical details for a single layer
- Testing a specific component or sensor
- Configuring hardware for a particular layer

---

## System Architecture (High-Level)

### Three-Layer Design

**Layer 1: The Brain** (`layer1-brain/`)
- **Hardware**: NVIDIA Jetson Orin Nano Super (67 TOPS, 1TB) - **ACQUIRED**
- **Functions**: Stereo vision, object detection, path planning, sensor fusion
- **AI Models**: YOLOv8n-INT8 @ 65 FPS, semantic segmentation @ 36 FPS
- **See**: `/layer1-brain/CLAUDE.md` for full technical specifications

**Layer 2: The Heart** (`layer2-heart/`)
- **Hardware**: Speeduino v0.4 ECM (STM32F407-based) - **Sourcing**
- **Functions**: 2-stroke engine management, CAN telemetry, TunerStudio tuning
- **See**: `/layer2-heart/CLAUDE.md` for ECM configuration details

**Layer 3: The Muscle** (`layer3-muscle/`)
- **Hardware**: Progressive Automations actuators + Dynamixel servo - **Sourcing**
- **Functions**: Steering, braking, throttle control with redundant sensors
- **See**: `/layer3-muscle/CLAUDE.md` for actuator specifications

**Communication Backbone**: CAN Bus 2.0B @ 500 kbps
**See**: `/integration/CLAUDE.md` for complete protocol specification

---

## Cross-Cutting Concerns (PM Responsibilities)

### 1. CAN Bus Integration (`integration/`)

**Network Architecture**:
```
[Jetson Orin]──────[CAN 500kbps]──────[Speeduino ECM]
    (120Ω)                                  (120Ω)
      │                                        │
  [Vision/Radar]                        [Actuators]
      │                                  ├─ Steering (0x200)
  [GPS/IMU]                              ├─ Brake (0x210)
                                         └─ Throttle (0x220)
```

**Key Specifications**:
- Protocol: CAN 2.0B (ISO 11898), 500 kbps
- Termination: 120Ω at both ends only
- Heartbeat: Jetson sends 0x500 every 100ms (fail-safe if >300ms)
- Message IDs: 0x001 (emergency), 0x100 (ECM), 0x200 (actuators), 0x500 (heartbeat)

**See**: `/integration/CLAUDE.md` for complete message definitions and wiring diagrams

### 2. Power Budget

**12V Electrical System**:
- Jetson Orin Nano: 5-7A typical, 15A peak (15-25W typical, 60W TDP)
- Steering Actuator: 12A peak (PA-04-12-400 @ 12V)
- Brake Actuator: 12A peak (PA-04-6-400 @ 12V)
- Throttle Servo: 5A peak (Dynamixel MX-106T)
- Radar + Cameras + Sensors: ~3A
- **Total Peak Draw**: ~47A (560W)
- **Battery**: 60Ah AGM (Optima D31A, $280) - supports 1-2 hours operation

**Power Management**:
- Relay-controlled actuator circuits (prevent Jetson GPIO overload)
- Voltage monitoring on CAN bus (ECM → Jetson)
- Low-battery fail-safe (return to safe state at <11.5V)

### 3. Safety Systems & Testing Coordination

**Critical Safety Mechanisms** (span all layers):
- Hardware emergency stop (killswitch) - overrides all software
- Redundant position sensors (Hall + potentiometer per actuator)
- CAN watchdog (heartbeat timeout triggers fail-safe)
- Vision system failure detection (FPS drop, object detection failures)
- Geofencing (software boundaries prevent leaving test area)
- Manual override capability

**6-Phase Testing Protocol**:
1. **Bench Testing**: Power, CAN communication, sensor validation, actuator strokes
2. **Static Engine Testing**: Speeduino startup, fuel system, ignition, autonomous throttle
3. **Tethered Testing**: Steering/brake actuation, low-speed movement, vision pipeline
4. **Low-Speed Autonomous** (5 mph): GPS waypoints, obstacle avoidance in open field
5. **Progressive Speed Increases**: 5→10→15→20 mph over multiple sessions
6. **Edge Cases**: Sensor failures, low battery, extreme cold (-20°C), CAN disconnect

**Success Criteria**:
- All sensors within expected ranges
- Actuators respond <50ms to commands
- Zero CAN bus errors over 1 hour
- Emergency stop <100ms response
- Vision system 60+ FPS at 720p
- Graceful degradation for single-point failures

**See**: Each layer's CLAUDE.md for subsystem-specific testing procedures

### 4. Centralized Bill of Materials (BOM)

**Complete System Cost: ~$3,981**

| Component | Model/Type | Directory | Status | Cost |
|-----------|------------|-----------|--------|------|
| Main Computer | Jetson Orin Nano Super (1TB) | `layer1-brain/` | **Acquired** | $599 |
| ECM | Speeduino v0.4 (STM32F407) | `layer2-heart/` | Sourcing | $150 |
| Steering Actuator | Progressive PA-04-12-400 | `layer3-muscle/` | Sourcing | $450 |
| Brake Actuator | Progressive PA-04-6-400 | `layer3-muscle/` | Sourcing | $380 |
| Throttle Servo | Dynamixel MX-106T | `layer3-muscle/` | Planned | $480 |
| Stereo Cameras | Arducam IMX477 Kit | `layer1-brain/` | Sourcing | $250 |
| Radar | TI AWR1843 (77 GHz) | `layer1-brain/` | Planned | $300 |
| GPS (RTK) | u-blox ZED-F9P | `layer1-brain/` | Planned | $200 |
| IMU | BNO085 9-DOF | `layer1-brain/` | Planned | $20 |
| CAN Controller | MCP2515 + MCP2551 | `integration/` | Planned | $15 |
| CAN Cable | Belden 3084A (20m) | `integration/` | Planned | $50 |
| Battery | 60Ah AGM (Optima D31A) | All layers | Planned | $280 |
| Mounting Hardware | Brackets, fasteners | `mechanical/` | Planning | $150 |
| Sensors (CLT, IAT, TPS) | Speeduino-compatible | `layer2-heart/` | Sourcing | $75 |
| Wideband O2 | Bosch LSU 4.9 + controller | `layer2-heart/` | Planned | $200 |
| Misc (wiring, connectors) | Various | All layers | Ongoing | $382 |

**BOM Management**:
- Update this table when procuring components
- Link to subdirectory CLAUDE.md for technical specifications
- Track status: Acquired → Sourcing → Planned → Obsolete

---

## Timeline & Milestones

### Phase 1: Foundation - **COMPLETE** ✅
- [x] Architecture design
- [x] Comprehensive technical research
- [x] Engineering-level website documentation (index.html ready for deployment)
- [x] Complete BOM with cost estimates
- [x] Project directory structure with subdirectory CLAUDE.md files
- [ ] Hardware procurement **IN PROGRESS**
  - [x] Jetson Orin Nano Super acquired
  - [ ] Speeduino ECM sourcing
  - [ ] Actuators sourcing
  - [ ] Vision sensors sourcing

### Phase 2: Core Systems - **Winter/Spring 2025**
- [ ] Jetson Orin Nano setup (JetPack 6.2, Ubuntu 22.04 L4T)
- [ ] Speeduino installation and baseline 2-stroke tuning
- [ ] CAN bus network implementation (wiring, termination, testing)
- [ ] Basic sensor integration (GPS, IMU, CAN communication validation)

### Phase 3: Vision & Perception - **Spring/Summer 2025**
- [ ] Stereo camera calibration (intrinsic/extrinsic parameters)
- [ ] Object detection model training (YOLOv8 on winter dataset)
- [ ] Radar integration (TI AWR1843 interfacing)
- [ ] Sensor fusion algorithms (EKF @ 100 Hz)

### Phase 4: Actuation & Control - **Summer 2025**
- [ ] Linear actuator mounting and stroke testing
- [ ] Steering system integration (linkage geometry, fail-safe)
- [ ] Brake-by-wire implementation (hydraulic pressure validation)
- [ ] Throttle control system (cable actuation, kill switch)

### Phase 5: Autonomy - **Summer/Fall 2025**
- [ ] Path planning algorithms (A* global + DWA local)
- [ ] Obstacle avoidance (real-time trajectory adjustment)
- [ ] GPS waypoint navigation (RTK precision)
- [ ] Autonomous mode testing (6-phase protocol)

### Phase 6: Polish & Safety - **Fall 2025**
- [ ] Fail-safe systems validation (all edge cases)
- [ ] Emergency stop protocols (hardware + software)
- [ ] Android app development (driver + developer modes)
- [ ] Field testing and refinement

---

## GitHub Workflow & Version Control (When Initialized)

**Repository Structure** (matches directory layout):
```
robosnomo/
├── .gitignore              # Large files, build artifacts, API keys
├── .gitattributes          # Git LFS for models, datasets >100MB
├── README.md               # Public-facing project overview (to be created)
├── LICENSE                 # Open-source license (TBD)
├── CLAUDE.md               # This PM-level file (project context)
│
├── website/                # Static site (Cloudflare Pages)
├── layer1-brain/           # Python code, AI models, vision pipeline
├── layer2-heart/           # Speeduino firmware, TunerStudio configs
├── layer3-muscle/          # Actuator control code, PID tuning
├── integration/            # CAN protocol headers, test scripts
├── mechanical/             # CAD models, fabrication notes
├── datasets/               # Training images, labels (Git LFS)
├── docs/                   # Datasheets, research papers
└── tools/                  # Utilities, testing scripts
```

**Branching Strategy** (to be established):
- `main` - Stable releases only
- `develop` - Integration branch for all features
- `feature/layer1-vision` - Feature branches per subsystem
- `feature/layer2-ecu` - Subsystem-specific work
- `hotfix/can-timeout` - Critical bug fixes

**Commit Guidelines**:
- Prefix commits with subsystem: `[layer1-brain] Add YOLOv8 inference script`
- Reference issues/PRs: `[integration] Fix CAN heartbeat timeout (#12)`
- Update subdirectory CLAUDE.md when making architectural changes

**Git LFS** (Large File Storage):
- Enable for: `*.engine`, `*.onnx`, `*.pth`, `*.jpg`, `*.png`, `*.mp4`
- Datasets >100MB stored with LFS

**GitHub Initialization Checklist** (when ready):
- [ ] Create repository (public or private)
- [ ] Add .gitignore (Python, C++, JetPack, TunerStudio artifacts)
- [ ] Initialize Git LFS
- [ ] Create README.md (public-facing overview)
- [ ] Add LICENSE file
- [ ] Push initial structure
- [ ] Set up GitHub Issues for task tracking

---

## Technology Stack (Cross-Platform)

**AI & Vision** (`layer1-brain/`):
- Python 3.x, PyTorch 2.x, TensorFlow 2.15, TensorRT
- OpenCV with CUDA, YOLOv8, DeepLabv3+
- NVIDIA JetPack SDK 6.2, ROS 2 Humble
- python-can, SocketCAN (Linux kernel CAN)

**Embedded Systems** (`layer2-heart/`):
- C++ (Speeduino firmware v202501.1)
- TunerStudio MS (ECM tuning)
- Arduino IDE / PlatformIO

**Integration** (`integration/`):
- SocketCAN (Linux), python-can library
- Wireshark (CAN protocol analysis)
- candump, cansend (debugging tools)

**Mobile** (future):
- Android Studio, Kotlin/Java
- MQTT or WebSocket for telemetry

**Documentation** (`website/`):
- Static HTML/CSS/JS (Cloudflare Pages)

---

## Key Resources & Community

**Community Support**:
- Speeduino Forums: speeduino.com
- NVIDIA Jetson Community: developer.nvidia.com/jetson
- Vintage Sleds (VS): Polaris restoration expertise

**Technical Documentation** (see `docs/`):
- Speeduino Wiki, Jetson Orin datasheets
- CAN Bus ISO 11898 standard
- Research papers on winter navigation

---

## Next Steps (Immediate Actions)

**PM-Level Tasks**:
1. Deploy website to Cloudflare Pages (`website/index.html`)
2. Initialize GitHub repository (when ready to start coding)
3. Track hardware procurement status (update BOM table above)
4. Coordinate CAN message ID allocation across all layers

**Subsystem Tasks** (delegate to subdirectories):
- `layer1-brain/`: Set up Jetson development environment (JetPack 6.2)
- `layer2-heart/`: Procure Speeduino ECM (STM32F407 variant)
- `layer3-muscle/`: Source Progressive Automations actuators, design mounting brackets
- `integration/`: Design CAN wiring schematic, order MCP2515 modules and Belden cable
- `mechanical/`: Assess Polaris Colt chassis, plan actuator mounting points
- `datasets/`: Begin winter obstacle dataset collection planning
- `tools/`: Set up CAN bus testing utilities (candump logger, cansend simulator)

---

## Notes for Collaboration

- This is a **hobby-plus project** - take safety seriously
- **Document everything** (photos, schematics, code comments)
- **Test incrementally** - never skip validation steps
- **Keep backups** of all configurations
- **Share learnings** with the community

Looking for contributors with expertise in:
- **Mechanics**: Vintage Polaris repair, chassis modification
- **Welding/Fabrication**: Actuator mounting, structural work
- **Software**: Python (AI), C++ (embedded), ROS 2
- **Electronics**: CAN bus, sensor integration, power systems

---

**Last Updated**: November 18, 2025
**Project Lead**: Anthony Slosar
**PM Status**: Phase 1 Complete → Phase 2 Hardware Procurement Starting
**Website**: `/website/index.html` (ready for Cloudflare deployment)

---

## Quick Navigation Reference

| To work on... | Navigate to... | For details on... |
|---------------|----------------|-------------------|
| Vision, AI, path planning | `layer1-brain/` | Jetson setup, YOLOv8, stereo cameras, sensor fusion |
| ECM, engine tuning | `layer2-heart/` | Speeduino config, 2-stroke setup, TunerStudio |
| Actuators, servos, linkages | `layer3-muscle/` | Progressive actuators, Dynamixel, fail-safes |
| CAN protocol, wiring | `integration/` | Message IDs, termination, testing, error handling |
| Chassis, mounting, welding | `mechanical/` | Polaris Colt restoration, fabrication |
| AI training, datasets | `datasets/` | Winter images, YOLOv8 training, TensorRT export |
| Datasheets, research papers | `docs/` | Component manuals, research notes |
| Testing scripts, utilities | `tools/` | CAN loggers, calibration, benchmarks |
| Public website | `website/` | Cloudflare deployment, documentation |
| **Project coordination** | **HERE (root)** | **BOM, timeline, cross-cutting concerns, GitHub** |
