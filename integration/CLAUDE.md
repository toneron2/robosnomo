# Integration - CAN Bus & System Testing
## Project RoboSnomo | Inter-Layer Communication

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope of this Subdirectory

This directory contains all work related to **system integration** - primarily CAN bus communication between layers, but also system-level testing and validation.

**Responsibilities**:
- CAN bus protocol definitions (message IDs, data formats, timing)
- Network architecture (topology, termination, wiring)
- Inter-layer communication (Jetson ↔ ECM ↔ Actuators)
- Integration testing (end-to-end scenarios)
- Fail-safe coordination (heartbeat, timeout, fault handling)

---

### CAN Bus Specifications

**Protocol**: CAN 2.0B (ISO 11898 standard)
**Bit Rate**: 500 kbps (optimal speed/reliability balance for automotive)
**Identifier Format**: 11-bit standard (2048 unique message IDs)
**Max Cable Length**: 100 meters @ 500 kbps with proper termination
**Worst-Case Latency**: <2ms for highest priority messages
**Bus Utilization Target**: <80% (maintain deterministic timing)

---

### Physical Layer

**Differential Signaling**:
- Recessive State (Logic 1): CAN-H = 2.5V, CAN-L = 2.5V, Differential = 0V
- Dominant State (Logic 0): CAN-H = 3.5V, CAN-L = 1.5V, Differential = 2.0V

**Wiring Requirements**:
- Cable: Shielded twisted pair, 120Ω characteristic impedance (Belden 3084A or DeviceNet)
- Twist rate: Minimum 40 twists/meter (ideally 1 twist per 25mm)
- Wire gauge: 20-24 AWG for automotive applications
- Termination: 120Ω resistors at **both ends** only (measure 60Ω total)
- Stub length: Keep actuator/sensor connections under 30cm maximum
- Shielding: Ground shield at ONE point only (prevent ground loops)

**Network Topology**:
```
[Jetson Orin Nano]────────[CAN BUS 500 kbps]────────[Speeduino ECM]
      (120Ω)                                              (120Ω)
        │                                                    │
        ├─── MCP2515 SPI-to-CAN                            │
        │                                                    │
   [Vision/Radar]                                   [Actuator Nodes]
        │                                            ├─ Steering (0x200)
        │                                            ├─ Brake (0x210)
  [GPS/IMU Module]                                   └─ Throttle (0x220)
```

---

### Message ID Architecture (Priority-Based)

Lower ID numbers = Higher priority (wins arbitration)

| ID Range | Subsystem | Priority | Message Type | Frequency |
|----------|-----------|----------|--------------|-----------|
| 0x001-0x0FF | Safety & Emergency | **Highest** | Emergency stop, Kill switch | Event-driven |
| 0x100-0x1FF | ECM Critical Data | **High** | Engine RPM, Coolant temp, IAT, TPS | 100 Hz (10ms) |
| 0x200-0x2FF | Actuator Commands | **High** | Steering, Brake, Throttle commands | 50 Hz (20ms) |
| 0x280-0x2FF | Actuator Feedback | **High** | Position, status, faults | 50 Hz (20ms) |
| 0x300-0x3FF | Vision/Sensor Data | **Medium** | Obstacle detection, GPS | 10-20 Hz |
| 0x400-0x4FF | Telemetry/Diagnostics | **Low** | Status messages, Logs | 1-10 Hz |
| 0x500-0x7FF | Heartbeat & Expansion | **Medium** | System heartbeat (0x500) | 10 Hz (100ms) |

---

### Message Definitions

#### 0x001: Emergency Stop (Any node → All nodes)
```
DLC: 1 byte
Data[0]: 0x00=emergency stop, 0x01=release
```

#### 0x100: ECM Telemetry (ECM → Jetson)
```
DLC: 8 bytes
Data[0-1]: RPM (big-endian, 0-16383)
Data[2]: Coolant temp (°C + 40 offset)
Data[3]: Intake air temp (°C + 40)
Data[4]: TPS (0-100%)
Data[5-6]: Battery voltage (0.1V resolution)
Data[7]: Status flags
```

#### 0x200: Steering Command (Jetson → Steering Actuator)
```
DLC: 8 bytes
Data[0-1]: Target position (mm)
Data[2-3]: Max speed (mm/s)
Data[4]: Control flags
Data[5-6]: Checksum
Data[7]: Reserved
```

#### 0x280: Steering Status (Steering Actuator → Jetson)
```
DLC: 8 bytes
Data[0-1]: Current position Hall (mm)
Data[2-3]: Current position Pot (mm)
Data[4]: Status (moving, fault, etc.)
Data[5]: Motor current
Data[6]: Temperature
Data[7]: Fault code
```

#### 0x500: Heartbeat (Jetson → All Actuators)
```
DLC: 1 byte
Data[0]: 0x01 (alive signal)
Period: 100ms
Timeout: >300ms triggers fail-safe
```

---

### Error Handling

**Five CAN Error Detection Mechanisms**:
1. Bit Error: Mismatch between transmitted and received bit
2. Stuff Error: >5 consecutive identical bits
3. CRC Error: Cyclic redundancy check fails
4. Form Error: Fixed-format fields contain illegal values
5. Acknowledgment Error: No node acknowledges reception

**Three Error States**:
- Error Active: Normal operation (TEC <128, REC <128)
- Error Passive: Degraded mode (128 ≤ TEC/REC <256)
- Bus-Off: Node disconnected (TEC ≥256), requires recovery

---

### Directory Structure

```
integration/
├── CLAUDE.md                 # This file
├── can-protocol/
│   ├── message_definitions.h
│   ├── can_db.dbc           # CAN database for analysis tools
│   └── protocol_spec.md
├── wiring/
│   ├── can_schematic.pdf
│   ├── pinout_diagrams/
│   └── termination_guide.md
├── testing/
│   ├── can_loopback_test.py
│   ├── bus_analyzer_logs/
│   └── integration_test_suite.py
└── tools/
    ├── candump_logger.sh
    ├── cansend_simulator.py
    └── wireshark_filters/
```

---

### Key References

- [Jetson CAN Setup](/storage/emulated/0/Documents/projects/robosnomo/layer1-brain/CLAUDE.md)
- [ECM CAN Interface](/storage/emulated/0/Documents/projects/robosnomo/layer2-heart/CLAUDE.md)
- [Actuator CAN Protocol](/storage/emulated/0/Documents/projects/robosnomo/layer3-muscle/CLAUDE.md)

---

**Last Updated**: November 18, 2025
**Status**: Phase 1 - Specification Complete (Implementation pending hardware)
