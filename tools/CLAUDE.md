# Tools - Utilities & Development Scripts
## Project RoboSnomo | Development Tooling

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope

Reusable scripts, testing utilities, and development tools used across sub-projects.

**Tools**:
- CAN bus testing (candump loggers, cansend simulators)
- Sensor calibration scripts
- Performance benchmarking (FPS, latency, power)
- Data collection automation
- Log analysis and visualization

**Directory Structure**:
```
tools/
├── CLAUDE.md
├── can-utils/
│   ├── can_logger.py          # Log CAN messages to SQLite
│   ├── can_replay.py          # Replay logged CAN sessions
│   └── can_monitor_gui.py     # Real-time CAN bus monitor
├── calibration/
│   ├── camera_calibration.py  # Stereo camera intrinsic/extrinsic
│   ├── sensor_calibration.py  # TPS, CLT, IAT calibration wizard
│   └── actuator_tuning.py     # PID tuning for actuators
├── benchmarks/
│   ├── vision_benchmark.py    # Measure FPS, latency of vision pipeline
│   ├── power_monitor.py       # Track Jetson power consumption
│   └── integration_test.py    # End-to-end system test
├── data-collection/
│   ├── image_capture.py       # Automated dataset collection
│   ├── gps_logger.py          # Log GPS trails for mapping
│   └── telemetry_recorder.py  # Record all CAN telemetry
└── visualization/
    ├── plot_can_logs.py       # Matplotlib CAN data visualization
    ├── 3d_pointcloud_viewer.py # Display stereo depth point clouds
    └── dashboard.html         # Real-time web dashboard
```

**Status**: Active (Tools added as development progresses)

---

**Last Updated**: November 18, 2025
