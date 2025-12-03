# Layer 3: The Muscle - Actuation Systems
## Project RoboSnomo | Steering, Braking, Throttle Control

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope of this Subdirectory

This directory contains all work related to **Layer 3: The Muscle** - the physical actuation systems that control steering, braking, and throttle.

**Responsibilities**:
- Steering actuation (linear actuator control, position feedback)
- Brake actuation (hydraulic pressure control, emergency stop response)
- Throttle control (cable pull mechanism, position accuracy)
- Redundant position sensing (Hall effect + potentiometer per actuator)
- CAN bus interface (receive commands from Jetson, report status)
- Fail-safe modes (heartbeat timeout, overcurrent protection, manual override)

**Hardware**:
- Steering: Progressive Automations PA-04-12-400 (400 lbs, 12", IP66)
- Braking: Progressive Automations PA-04-6-400 (400 lbs, 6")
- Throttle: Dynamixel MX-106T (84 kg·cm, RS-485, 12-bit encoder)
- Linkages: Clevis mounts, spherical bearings (Aurora COM-series, QA1 Precision)
- Power: 12V system, ~37A peak draw

---

### Technical Specifications

#### Steering Actuator: Progressive Automations PA-04-12-400

**Force Rating**: 400 lbs (1780 N) push/pull
**Stroke Length**: 12 inches (305 mm total travel)
**Speed**: 20-30 mm/s (responsive navigation control)
**Voltage**: 12V DC (matches snowmobile electrical system)
**Current Draw**: 10-12A peak, 6-8A typical operation
**IP Rating**: IP66 (dust-tight, water jet resistant)
**Temperature Range**: -26°C to +65°C (adequate for most winter use)
**Position Feedback**: Built-in potentiometer (0-5V analog output)
**Cost**: ~$450

**Control Interface**: Analog (0-5V or 0-10V position command) or PWM

**Professional Upgrade Option**: LINAK LA77 or TiMOTION MA2
- Force: 6000-10000N (1350-2250 lbs)
- IP69K rated (high-pressure steam cleaning resistant)
- CAN bus / SAE J1939 integration
- Temperature: -40°C to +85°C (extreme cold rated)
- Cost: $600-1,500

#### Brake Actuator: Progressive Automations PA-04-6-400

**Force Rating**: 400 lbs (1780 N) - adequate for hydraulic brake actuation
**Stroke Length**: 6 inches (152 mm travel for full brake range)
**Speed**: 40-50 mm/s (fast response for emergency stops)
**Response Time**: <100ms from CAN command to actuation
**Target Performance**: 1200 PSI brake pressure (requires ~530 lbf calculated force)

**Note**: 400 lb actuator is adequate for most scenarios; 800-1000 lb provides safety margin for emergency stops on ice.

**Fail-Safe Option**: Spring-Applied Hydraulic Release (SAHR) brake system
- Automatic braking on power loss (fail-safe state)
- Spring applies brake; hydraulic pressure releases
- Critical for autonomous operation safety

#### Throttle Control: Dynamixel MX-106T

**Torque**: 84 kg·cm (8.4 N·m holding torque) - more than adequate for cable pull
**Position Resolution**: 12-bit absolute encoder (4096 positions, 0.088° resolution)
**Communication**: RS-485 (daisy-chain multiple servos)
**Feedback**: Full telemetry (position, velocity, current, temperature)
**Operating Voltage**: 12V (11-14.8V range)
**Current Draw**: 2-5A peak
**Temperature Range**: -5°C to +80°C (**requires heated enclosure for extreme cold**)
**Cost**: ~$480

**Budget Alternative**: Hitec HS-7950TH RC Servo
- Torque: 32 kg·cm (sufficient for cable pull)
- Standard RC PWM interface
- Cost: $100-120
- Weatherproof with proper enclosure
- No built-in feedback (requires separate position sensor)

**Throttle Cable Requirements**:
- Estimated pull force: 50-150N
- Cable travel: 38-50mm typical
- Integration with kill switch (hardware-level engine shutdown)

---

### Control Interfaces

| Interface | Best For | Advantages | Disadvantages |
|-----------|----------|------------|---------------|
| **CAN Bus / CANopen** | Professional multi-actuator system | Position control, diagnostics, standardized protocol | Requires CAN-enabled actuators (higher cost) |
| **RS-485** | Dynamixel servo motors | Long-distance, robust, multiple devices on one bus | Requires Dynamixel protocol library |
| **PWM** | Budget RC servos, proof-of-concept | Simple, widely supported | No built-in feedback (requires separate sensors) |
| **Analog (0-5V or 0-10V)** | Industrial actuators without digital interface | Simple wiring, reliable | No protocol-based diagnostics |

**Recommendation for Project Aurora**: Mixed approach
- Linear actuators: Analog control (0-5V) + separate CAN bus for telemetry
- Throttle servo: RS-485 (Dynamixel) or PWM (RC servo) + separate position sensor

---

### Redundant Position Sensing

**Critical Safety Feature**: Dual sensors per actuator prevent runaway on single sensor failure.

**Sensor Combination**:
1. **Hall Effect Encoder**: Digital, high resolution, immune to EMI
2. **Analog Potentiometer**: Continuous 0-5V output, simple, reliable

**Cross-Check Logic**:
- If sensors disagree by >5%, flag fault and enter limp mode
- Stop actuator movement, engage brake, alert operator
- Prevents catastrophic failure due to sensor malfunction

**Implementation**:
```python
# Pseudocode for redundant sensor check
hall_position = read_hall_encoder()  # 0-4096 counts
pot_position = read_potentiometer()  # 0-1023 ADC counts

# Convert to same units (e.g., millimeters)
hall_mm = hall_position * stroke_length / 4096
pot_mm = pot_position * stroke_length / 1023

error = abs(hall_mm - pot_mm)
if error > 5.0:  # 5mm threshold
    raise SensorFault("Position mismatch: Hall={hall_mm}, Pot={pot_mm}")
    enter_fail_safe_mode()
```

---

### Power Requirements

**Peak Current Draw (All Systems)**:
- Steering actuator: 10-12A peak
- Brake actuator: 10-12A peak
- Throttle servo: 2-5A peak
- Jetson Orin Nano: 5A typical (25W MaxN mode @ 12V with DC-DC converter)
- **Total System**: ~37A peak draw

**Battery Recommendations**:
- **Budget**: 60Ah AGM battery (Optima YellowTop D31A, $250-300)
  - Runtime: ~1.5 hours at 50% average load
- **Performance**: 50Ah LiFePO4 with heating blanket ($600-800)
  - Better cold-weather performance, lighter weight, longer cycle life

**Cold Weather Considerations**:
- Battery capacity drops 20-40% at -20°C
- Size battery accordingly (60Ah AGM → effective 36-48Ah in extreme cold)
- LiFePO4 requires heating blanket below 0°C for charging

**Wiring**:
- Main power: 8-10 AWG wire from battery to distribution block
- Actuator power: 12-14 AWG (10-12A circuits)
- Fuses: 15A-30A automotive blade fuses per circuit
- Kill switch: Hardware relay in ignition circuit (independent of software)

---

### Mounting & Mechanical Integration

**Linkage Design**:
- **Type**: Clevis (dual-pivot) mounting most common for linear actuators
- **Rod Ends**: Quality spherical bearings (Aurora COM-series, QA1 Precision) minimize backlash
- **Backlash Target**: <5mm at ski for steering precision
- **Mechanical Advantage**: Optimize linkage geometry to reduce actuator force requirements

**Mounting Considerations**:
- Vibration damping: Rubber bushings at pivot points
- Electronics isolation: Lord Micromounts or similar vibration isolators
- Materials: Aluminum 6061-T6 (lightweight, corrosion-resistant), stainless steel for high-stress applications
- Welding: TIG welding for aluminum, MIG for steel (see `/mechanical/` for fabrication details)

**Steering Linkage Calculations** (Example):
- Ski movement required: ±30° (typical snowmobile range)
- Linkage ratio: 3:1 (actuator moves 3" for 10° ski movement)
- Actuator stroke needed: 9" (12" actuator provides margin)
- Force at ski: 150 lbs → force at actuator: 50 lbs (well within 400 lb rating)

**See** `/mechanical/CLAUDE.md` for detailed fabrication procedures and mounting bracket designs.

---

### CAN Bus Communication

#### Commands from Jetson (50 Hz = 20ms period)

**Steering Command**:
```
ID: 0x200
DLC: 8 bytes
Data[0-1]: Target position (mm or 0.01" units, big-endian)
Data[2-3]: Maximum speed (mm/s)
Data[4]: Control flags (enable, emergency stop, manual override)
Data[5-6]: Checksum or sequence counter
Data[7]: Reserved
```

**Brake Command**:
```
ID: 0x210
DLC: 8 bytes
Data[0-1]: Target brake pressure or position
Data[2]: Braking mode (0=normal, 1=emergency, 2=hold)
Data[3-7]: Reserved / checksum
```

**Throttle Command** (mirrored from ECM or direct):
```
ID: 0x220 (see layer2-heart for details)
DLC: 8 bytes
Data[0-1]: Target throttle position (0-1000, 0.1% resolution)
Data[2]: Command mode (0=manual, 1=autonomous)
...
```

#### Status Feedback to Jetson (50 Hz)

**Steering Status**:
```
ID: 0x280
DLC: 8 bytes
Data[0-1]: Current position (mm, Hall encoder)
Data[2-3]: Current position (mm, potentiometer)
Data[4]: Status flags (moving, stalled, limit reached, fault)
Data[5]: Motor current (scaled 0-255)
Data[6]: Temperature (°C offset by 40)
Data[7]: Fault code (0=OK, 1=sensor mismatch, 2=overcurrent, etc.)
```

**Heartbeat Monitoring**:
- Jetson sends 0x500 heartbeat every 100ms
- If actuator receives no heartbeat for >300ms:
  - Stop all actuator movement
  - Engage brake
  - Center steering (return to neutral)
  - Throttle to idle
  - Enter fail-safe mode until heartbeat resumes

**See** `/integration/CLAUDE.md` for complete CAN protocol specification.

---

### Fail-Safe Systems

**Critical Safety Features**:

1. **Heartbeat Timeout** (300ms):
   - If no 0x500 message from Jetson, enter fail-safe
   - Prevents runaway if AI system crashes

2. **Overcurrent Protection**:
   - Monitor motor current; stop if >120% rated current for >5 seconds
   - Indicates mechanical binding or actuator overload

3. **Sensor Redundancy**:
   - Dual position sensors (Hall + pot)
   - Cross-check on every read; fault if mismatch >5mm

4. **Manual Override**:
   - Retain manual brake lever (mechanical backup, not electric)
   - Retain manual throttle cable (parallel to servo, selectable via switch)

5. **Hardware Kill Switch**:
   - Independent of all software/CAN bus
   - Latching relay wired directly to ignition circuit
   - Physical button + wireless remote backup

---

### Directory Structure (When Populated)

```
layer3-muscle/
├── CLAUDE.md                   # This file (domain knowledge)
├── actuator-control/
│   ├── steering_controller.py
│   ├── brake_controller.py
│   ├── throttle_servo.py
│   └── position_feedback.py
├── can-interface/
│   ├── actuator_can_messages.py
│   ├── heartbeat_monitor.py
│   └── fail_safe_logic.py
├── linkage-design/
│   ├── steering_geometry.py  # Calculate linkage ratios
│   ├── force_calculations.py
│   └── cad_models/          # SolidWorks/Fusion360 designs
├── tests/
│   ├── bench_test_actuators.py
│   ├── stroke_validation.py
│   ├── fail_safe_scenarios.py
│   └── can_loopback_test.py
├── firmware/
│   ├── arduino_actuator_bridge/  # If using Arduino to bridge CAN→PWM
│   └── dynamixel_config/
└── docs/
    ├── progressive_pa04_datasheet.pdf
    ├── dynamixel_mx106_manual.pdf
    └── mounting_procedures.md
```

---

### Key References

**Root-Level Documentation**:
- [Project RoboSnomo CLAUDE.md](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md) - PM-level overview
- [Actuator Research Notes](/storage/emulated/0/Documents/projects/robosnomo/docs/actuator-research.md) - Detailed component comparisons

**Integration Points**:
- `/integration/CLAUDE.md` - CAN bus protocols, message timing, fail-safe specifications
- `/layer1-brain/CLAUDE.md` - Jetson sends steering/brake/throttle commands
- `/layer2-heart/CLAUDE.md` - ECM may mirror throttle commands or provide engine protection limits
- `/mechanical/CLAUDE.md` - Mounting brackets, linkage fabrication, welding procedures

**External Resources**:
- Progressive Automations: https://www.progressiveautomations.com/
- LINAK Actuators: https://www.linak.com/
- Dynamixel Manual: https://emanual.robotis.com/docs/en/dxl/mx/mx-106/
- Aurora Rod Ends: https://www.aurorabearing.com/

---

### Development Priorities

**Phase 1: Procurement & Design**
- [ ] Source Progressive Automations PA-04-12-400 steering actuator
- [ ] Source Progressive Automations PA-04-6-400 brake actuator
- [ ] Order Dynamixel MX-106T throttle servo (or Hitec budget alternative)
- [ ] Order spherical bearings and clevis mounts
- [ ] Design linkage geometry in CAD (steering, brake, throttle)

**Phase 2: Bench Testing**
- [ ] Test actuator full stroke (extension/retraction)
- [ ] Verify position feedback accuracy (Hall + pot)
- [ ] Measure current draw at various loads
- [ ] Test fail-safe modes (heartbeat timeout, overcurrent)

**Phase 3: CAN Bus Integration**
- [ ] Implement CAN message handlers (0x200 steering, 0x210 brake, 0x220 throttle)
- [ ] Implement status feedback (0x280 steering status, etc.)
- [ ] Test heartbeat monitoring (disconnect Jetson, verify fail-safe)

**Phase 4: Mechanical Integration**
- [ ] Fabricate mounting brackets (see `/mechanical/`)
- [ ] Install actuators on chassis with proper alignment
- [ ] Connect linkages to steering, brake, throttle
- [ ] Verify mechanical advantage and force requirements

**Phase 5: Static Testing**
- [ ] Test steering actuation (stationary, verify ski movement)
- [ ] Test brake actuation (measure brake pressure with gauge)
- [ ] Test throttle cable pull (verify full range 0-100%)
- [ ] Validate redundant sensors (cross-check logic)

**Phase 6: Dynamic Testing**
- [ ] Tethered testing (10m rope, limited movement)
- [ ] Low-speed autonomous testing (5 mph, verify steering response)
- [ ] Emergency stop testing (measure stopping distance)

---

### Entry/Exit Process

**When Working in this Subdirectory**:
1. Review this CLAUDE.md to understand actuation scope and safety requirements
2. Check `/integration/CLAUDE.md` for CAN bus message formats and timing
3. Coordinate with `/mechanical/` for mounting bracket fabrication
4. Work on actuator control code, linkage design, or testing
5. Update this CLAUDE.md with force calculations, mounting notes, test results

**Returning to PM Level**:
- Navigate to `/storage/emulated/0/Documents/projects/robosnomo/`
- Read root CLAUDE.md for project-wide status
- Coordinate inter-layer integration via `/integration/`

---

**Last Updated**: November 18, 2025
**Subsystem Lead**: (TBD)
**Status**: Phase 1 - Procurement (Sourcing Progressive Automations actuators)
