# Layer 1: The Brain - AI & Perception
## Project RoboSnomo | Autonomous Vision & Decision Making

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope of this Subdirectory

This directory contains all work related to **Layer 1: The Brain** - the AI and perception system that enables autonomous navigation.

**Responsibilities**:
- Stereo vision processing and depth estimation
- Object detection and obstacle recognition
- Semantic segmentation (terrain classification)
- Sensor fusion (vision + radar + GPS + IMU)
- Path planning and navigation algorithms
- High-level decision making

**Hardware**:
- NVIDIA Jetson Orin Nano Super (1TB) - **ACQUIRED**
- Arducam IMX477 Stereo Camera Kit (12.3MP, HDR)
- Texas Instruments AWR1843 Radar (77 GHz)
- u-blox ZED-F9P GPS (RTK, <2cm accuracy)
- BNO085 9-DOF IMU

---

### Technical Specifications

#### Compute Platform: NVIDIA Jetson Orin Nano Super

**AI Performance**:
- 67 TOPS (INT8 sparse inference, Super Mode)
- 1,024 CUDA cores, 32 Tensor cores (Ampere architecture)
- 6-core ARM Cortex-A78AE CPU @ 1.7 GHz
- 8 GB LPDDR5 RAM (102 GB/s bandwidth)
- 1 TB NVMe SSD storage

**Camera Interfaces** (MIPI CSI-2):
- CAM0: 1x 2-lane CSI
- CAM1: 1x 4-lane CSI (configurable as 2x 2-lane)
- Supports up to 6 simultaneous camera streams
- ISP 6.x with hardware-accelerated debayering, HDR, noise reduction

**I/O**:
- USB: 1x USB-C 3.2 + 4x USB-A 3.2 (10 Gbps)
- Ethernet: Gigabit (10/100/1000 Mbps)
- GPIO: 40-pin header (I2C, SPI, UART, PWM)
- DisplayPort 1.2 output (4K @ 30 Hz)

**Power**:
- 25W MaxN/Super Mode, configurable 7-25W profiles
- Operates -25°C to +90°C (suitable for snowmobile winter use)

**Software**:
- JetPack SDK 6.2 (Ubuntu 22.04 L4T)
- CUDA 12.6, cuDNN, cuBLAS, TensorRT
- PyTorch 2.x, TensorFlow 2.15
- OpenCV with CUDA acceleration
- ROS 2 Humble for robotics middleware

**CAN Bus Integration**:
- MCP2515 SPI-to-CAN controller + MCP2551 transceiver
- SocketCAN Linux driver
- python-can library for application layer
- Connects to 500 kbps CAN 2.0B network (see `/integration/` for protocols)

---

#### Vision System: Arducam IMX477 Stereo Kit

**Sensor**: Sony IMX477 (12.3 MP backside illuminated)
**Resolution**: 1080p @ 240 FPS, 4K @ 60 FPS max
**HDR Mode**: 60 FPS HDR (critical for high-contrast snow environments)
**Baseline**: 20-30 cm (effective depth range: 2-25 meters)
**Synchronization**: Hardware sync board included in kit
**Cost**: ~$250

**Vision Processing Pipeline**:
1. Synchronized stereo frame capture (60 FPS @ 720p for real-time)
2. Image rectification (align epipolar lines using calibration matrix)
3. Stereo matching: SGBM (Semi-Global Block Matching) → disparity map
4. Depth estimation: depth = baseline × focal_length / disparity
5. 3D point cloud generation
6. Integration with object detection and segmentation

**Performance**: 44-78ms end-to-end latency (1080p), 60+ FPS @ 720p

---

#### AI Models

**Object Detection: YOLOv8n-INT8** (Recommended)
- FPS on Jetson Orin Nano: 65 FPS
- Power consumption: 7.4W (most energy-efficient)
- Accuracy loss: -3% mAP vs FP32 (minimal degradation)
- Custom training: Retrain on winter dataset (trees, rocks, trails, snow, other snowmobiles)
- Use RSOD (Remote Sensing Object Detection) dataset as starting point

**Semantic Segmentation: MFA-DeepLabv3+**
- FPS on Jetson Orin Nano: 36.4 FPS (real-time capable)
- Purpose: Pixel-wise terrain classification (snow, ice, rock, vegetation, trail, sky)
- Integration: Combines with stereo depth for 3D terrain understanding
- Training: Cityscapes dataset augmented with winter off-road images

**TensorRT Optimization**:
1. Train model in PyTorch/TensorFlow
2. Export to ONNX format
3. Convert to TensorRT engine with INT8 calibration
4. Benchmark: measure FPS and accuracy on Jetson
5. Deploy: load TensorRT engine in inference code

**Performance Gains**: 3-15x speedup (FP32 → FP16 → INT8), 16% power reduction

---

#### Path Planning Algorithms

**Hybrid Approach** (Recommended):
- **Global Planner**: A* algorithm on GPS waypoint map (1 Hz, generates optimal route)
- **Local Planner**: DWA (Dynamic Window Approach) for real-time obstacle avoidance (10-20 Hz)
- **Rationale**: A* provides optimal long-range path; DWA handles dynamic obstacles from vision

**Alternative: RRT (Rapidly-exploring Random Tree)**
- Speed: 0.23s planning time (very fast)
- Trade-off: Sub-optimal paths (longer routes)
- Use case: Fallback when A* takes too long in complex environments

---

#### Sensor Fusion

**Fusion Architecture**: Extended Kalman Filter (EKF) @ 100 Hz

**Integrated Sensors**:
1. **Stereo Vision** (20 Hz): 3D obstacle positions, terrain classification
2. **77 GHz Radar** (10 Hz): Long-range detection (200m), penetrates snow/fog
3. **IMU BNO085** (100 Hz): Acceleration, gyroscope, magnetometer
4. **GPS ZED-F9P RTK** (10 Hz): High-precision positioning (<2cm accuracy with RTK correction)

**Software**: ROS 2 `robot_localization` package
**Output**: Fused pose estimate at 100 Hz

**Radar Specifications (TI AWR1843)**:
- Frequency: 77 GHz (20x better range resolution than 24 GHz)
- Range: 200-300 meters
- Range resolution: 4 cm (vs 75 cm for 24 GHz)
- Snow/fog performance: Radar >> LiDAR > Camera in adverse weather
- Interface: UART/SPI to Jetson
- Cost: ~$300 (development kit)

---

### Winter-Specific Challenges & Solutions

| Challenge | Impact | Solution |
|-----------|--------|----------|
| Low contrast (white snow) | Stereo matching struggles | HDR imaging (IMX477 60 FPS HDR), radar fusion |
| Sun reflections | Overexposure, false detections | HDR capture, polarizing filters, adaptive exposure |
| Hidden obstacles (under snow) | Vision cannot detect buried rocks | Radar penetrates snow, GPS trail map overlay |
| Variable lighting (sunrise/sunset) | Inconsistent detection | Train on diverse lighting, normalized preprocessing |
| Falling snow/fog | Obscures vision | Radar primary sensor, reduce speed in degraded mode |

---

### CAN Bus Communication

**Interface to System**:
- Jetson transmits: Steering commands (0x200), Brake commands (0x210), Throttle commands (0x220)
- Jetson receives: Engine RPM (0x100), temperatures (0x110), battery voltage, fault codes
- Heartbeat: Send 0x500 every 100ms (actuators enter fail-safe if >300ms timeout)

**See** `/integration/CLAUDE.md` for complete CAN bus protocol details.

---

### Directory Structure (When Populated)

```
layer1-brain/
├── CLAUDE.md                # This file (domain knowledge)
├── vision/
│   ├── stereo_calibration.py
│   ├── depth_estimation.py
│   └── camera_drivers/
├── detection/
│   ├── yolov8_training.py
│   ├── tensorrt_export.py
│   └── inference.py
├── segmentation/
│   ├── deeplabv3_training.py
│   └── terrain_classifier.py
├── fusion/
│   ├── ekf_fusion.py
│   ├── sensor_sync.py
│   └── ros2_integration/
├── planning/
│   ├── astar_global.py
│   ├── dwa_local.py
│   └── path_executor.py
├── tests/
│   ├── test_vision_pipeline.py
│   ├── test_object_detection.py
│   └── benchmark_performance.py
└── configs/
    ├── camera_params.yaml
    ├── yolo_config.yaml
    └── fusion_params.yaml
```

---

### Key References

**Root-Level Documentation**:
- [Project RoboSnomo CLAUDE.md](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md) - PM-level overview
- [Complete BOM](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md#hardware-status) - Centralized parts list
- [Vision Research Notes](/storage/emulated/0/Documents/projects/robosnomo/docs/vision-ai-research.md) - Detailed research

**Integration Points**:
- `/integration/CLAUDE.md` - CAN bus protocols, message IDs, inter-layer communication
- `/layer2-heart/CLAUDE.md` - ECM telemetry (RPM, temps for decision making)
- `/layer3-muscle/CLAUDE.md` - Actuator control interface
- `/datasets/` - Training images, AI models, TensorRT engines

**External Resources**:
- NVIDIA Jetson Orin Docs: https://developer.nvidia.com/jetson-orin-nano-super-developer-kit
- YOLOv8 Ultralytics: https://docs.ultralytics.com/
- TensorRT Optimization: https://docs.nvidia.com/deeplearning/tensorrt/
- ROS 2 Humble: https://docs.ros.org/en/humble/

---

### Development Priorities

**Phase 1: Setup & Validation** (Current)
- [ ] Install JetPack 6.2 on Jetson Orin Nano
- [ ] Set up MCP2515 CAN controller (device tree overlay)
- [ ] Calibrate stereo cameras (intrinsic + extrinsic parameters)
- [ ] Benchmark YOLOv8n inference (measure FPS, latency, power)

**Phase 2: Vision Pipeline**
- [ ] Implement stereo rectification and SGBM depth estimation
- [ ] Train YOLOv8 on winter obstacle dataset (15,000+ images)
- [ ] Convert trained model to TensorRT INT8 format
- [ ] Integrate radar sensor (UART/SPI drivers)

**Phase 3: Sensor Fusion**
- [ ] Configure ROS 2 robot_localization package
- [ ] Implement EKF fusion (vision + radar + GPS + IMU)
- [ ] Validate fused pose accuracy (compare to ground truth GPS)

**Phase 4: Path Planning**
- [ ] Implement A* global planner on GPS waypoint map
- [ ] Implement DWA local planner with vision obstacle avoidance
- [ ] Integrate path planner output with CAN bus steering/throttle commands

**Phase 5: Field Testing**
- [ ] Test vision pipeline in outdoor winter conditions
- [ ] Validate obstacle detection accuracy (rocks, trees, other sleds)
- [ ] Measure performance degradation in snow/fog (switch to radar mode)

---

### Entry/Exit Process

**When Working in this Subdirectory**:
1. Review this CLAUDE.md to understand Layer 1 scope
2. Check `/integration/CLAUDE.md` for CAN bus interface contracts
3. Reference `/docs/vision-ai-research.md` for detailed technical background
4. Work on vision, detection, fusion, or planning code
5. Update this CLAUDE.md with discoveries, optimizations, or design decisions
6. When crossing boundaries (e.g., actuator control), return to root PM level

**Returning to PM Level**:
- Navigate to `/storage/emulated/0/Documents/projects/robosnomo/`
- Read root CLAUDE.md for project-wide status and next steps
- Coordinate integration work across layers via `/integration/`

---

**Last Updated**: November 18, 2025
**Subsystem Lead**: (TBD)
**Status**: Phase 1 - Setup & Validation (Jetson Orin Nano acquired, awaiting camera kit)
