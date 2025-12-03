# Layer 2: The Heart - Engine Control Module
## Project RoboSnomo | 2-Stroke Engine Management

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope of this Subdirectory

This directory contains all work related to **Layer 2: The Heart** - the engine control module (ECM) that manages the vintage 2-stroke engine.

**Responsibilities**:
- Fuel injection control (pulse width, timing)
- Ignition timing control (advance maps, dwell)
- Engine monitoring (RPM, temperatures, throttle position)
- CAN bus telemetry to AI layer (100 Hz)
- Autonomous throttle control (receive commands from Jetson via CAN)
- Engine protection (over-temp, over-rev limits)

**Hardware**:
- Speeduino v0.4 ECM (STM32F407-based **RECOMMENDED**)
- NTC thermistor sensors (CLT, IAT)
- 3-wire TPS (throttle position sensor)
- Hall effect or VR crank position sensor
- Wideband O2 sensor + controller (AEM 30-0300 X-Series)
- High-Z fuel injector (12-14Ω, sized for engine displacement)

---

### Technical Specifications

#### Speeduino v0.4 ECM (STM32F407 Variant)

**Microcontroller**: STM32F407VGT6
- Architecture: ARM Cortex-M4F @ 168 MHz
- Flash memory: 1 MB (4x larger than Arduino Mega)
- SRAM: 192 KB (24x larger than Mega)
- **Native dual CAN bus** (CAN0 + CAN1) - critical for this project

**Why STM32F407 over Arduino Mega**:
- Arduino Mega 2560 (ATmega2560) has **no native CAN** - requires external MCP2515 module
- STM32F407 has dual CAN controllers built-in
- 10x faster processor (168 MHz vs 16 MHz)
- 24x more RAM for complex algorithms
- Better for future expansion

**Input Capabilities**:
- Analog inputs: 16 channels, 0-5V, 10-bit resolution
- Supported sensors: CLT, IAT, TPS, MAP, O2 (narrowband/wideband)
- Trigger inputs: RPM1 (crank), RPM2 (cam)
  - VR sensors: Requires optional MAX9926 VR conditioner circuit
  - Hall effect / optical: Direct digital input (0-5V or 0-12V)

**Output Capabilities**:
- **Injector outputs**: 4 channels @ 2A each (High-Z injectors only, >8Ω)
- **Ignition outputs**: 4 channels, 5V or 12V logic level (JP1 jumper selectable)
  - For CDI trigger (common on 2-strokes): Use 5V logic output
- **Auxiliary outputs**: 4x medium-current (3A), 5x low-current (proto section)

**CAN Bus**:
- CAN0 pins: PD0 (RX), PD1 (TX)
- CAN1 pins: PB5 (RX), PB6 (TX)
- Requires external CAN transceiver (MCP2551 or TJA1050)
- OBD2 protocol support for standard PIDs
- Custom message definitions for autonomous control

**Power Requirements**:
- Input voltage: 12V nominal (automotive standard)
- Current draw: Varies with active outputs, typically <5A

**Firmware**: Speeduino v202501.1 (latest as of Nov 2025)

---

### 2-Stroke Engine Configuration

**Challenge**: Speeduino is optimized for 4-stroke engines. Single-cylinder 2-strokes require configuration workarounds.

#### Recommended Configuration

**Problem**: Most single-cylinder 2-strokes provide 1 trigger pulse per crankshaft revolution (360°). Speeduino's "Basic Distributor" mode expects 1 pulse per 720°, causing RPM to read at 2x actual speed.

**Solution**: Configure as a 2-cylinder engine

**TunerStudio Settings**:
- **Displacement**: Enter *half* of actual engine displacement
  - Example: 200cc actual engine → enter 100cc
- **Cylinders**: Set to **2** (even though it's single-cylinder)
- **Stroke**: 2-stroke mode
- **Injectors**: 1 (single injector)
- **Spark Mode**: Single channel or wasted spark
- **Load Algorithm**: **Alpha-N** (TPS-based, NOT Speed Density)

**Why Alpha-N for 2-Strokes**:
- Expansion chambers create erratic intake manifold pressure pulses
- MAP sensor readings are unreliable for load calculation
- TPS (throttle position) provides more predictable fueling
- Better for highly-tuned engines with tuned pipes

---

### Fuel & Ignition Management

#### Fuel Mapping (VE Table)

**Resolution**: 16x16 grid (256 cells total)
- **X-axis**: RPM (16 points, user-configurable range, e.g., 1000-8000 RPM)
- **Y-axis**: Load (16 points)
  - For Alpha-N: TPS % (0-100%)
  - For Speed Density: MAP kPa
- **VE values**: 0-255 (percentage of theoretical air volume)
- **Interpolation**: 3D interpolated for smooth transitions between cells

**Fuel Corrections**:
- Coolant temperature correction: Adds enrichment during warm-up
- Air temperature correction: Compensates for IAT changes
- Acceleration enrichment: Detects rapid TPS changes, adds fuel
- Cranking enrichment: Extra fuel during startup

#### Ignition Timing Control

**Advance Table**: 16x16 grid matching fuel table axes
- **Range**: Typically -10° to +40° BTDC (user-configurable)
- **Resolution**: Interpolated between points (1-5° effective resolution)

**Timing Corrections**:
- Coolant temperature: Adds/subtracts advance based on CLT
- Air temperature: Compensates for IAT changes
- Idle timing control: Closed-loop adjustment for idle stability
- Cranking advance: Fixed advance during startup (5-10° BTDC typical)

**Dwell Control**:
- **Fixed dwell**: Single value across all RPM/load
- **Variable dwell map**: 2D table (RPM vs load) for optimized spark energy
- **Voltage compensation**: Adjusts dwell based on battery voltage
- **Over-dwell protection**: Monitors coil charge time, cuts output if threshold exceeded

#### CDI Integration (for Vintage 2-Stroke)

Most vintage snowmobiles use **CDI (Capacitor Discharge Ignition)** instead of inductive coils.

**CDI Characteristics**:
- Requires trigger pulse, not dwell control
- High-energy spark (ideal for 2-strokes with large gaps)
- Fast rise time

**Speeduino CDI Connection**:
- Set JP1 jumper to **5V** output mode
- Use Speeduino ignition output as CDI trigger
- Speeduino controls timing; CDI provides spark energy
- Retains stock CDI reliability while gaining programmable timing control

---

### Sensor Configuration

#### Required Sensors

| Sensor | Type | Voltage | Purpose | Connection |
|--------|------|---------|---------|------------|
| CLT (Coolant Temp) | NTC Thermistor | 0-5V analog | Engine protection, warmup enrichment | Speeduino CLT input |
| IAT (Intake Air Temp) | NTC Thermistor | 0-5V analog | Density compensation, fuel corrections | Speeduino IAT input |
| TPS (Throttle Position) | 3-wire Pot | 0-5V analog | Alpha-N load source, accel detection | Speeduino TPS input |
| RPM/Crank Position | Hall or VR | 0-5V/0-12V digital | Engine speed, ignition timing reference | RPM1 input |
| O2 Sensor (Optional) | Wideband Controller | 0-5V analog | Closed-loop tuning, autotune | O2 input |

**Thermistor Calibration**:
- Speeduino supports 32-point temperature curves
- Default bias resistor: 2490Ω (standard for automotive NTC sensors)
- Fault defaults: CLT → 80°C, IAT → 20°C

**TPS Calibration**:
- Low voltage at closed throttle (typically 0.5-1.0V)
- High voltage at WOT (typically 4.0-4.5V)
- Calibrate in TunerStudio: set closed and WOT values

---

### CAN Bus Communication

#### Telemetry to Jetson (100 Hz)

**OBD2 Standard PIDs**:
- 0x0C: Engine RPM
- 0x05: Coolant temperature
- 0x0F: Intake air temperature
- 0x11: Throttle position
- 0x42: Battery voltage
- Custom: Injector duty cycle, error codes

**CAN Message Format** (Example):
```
ID: 0x100 (ECM Critical Data)
DLC: 8 bytes
Data[0-1]: RPM (big-endian, 0-16383 range)
Data[2]: Coolant temp (°C + 40 offset, 0-255)
Data[3]: Intake air temp (°C + 40 offset)
Data[4]: TPS (0-100%)
Data[5-6]: Battery voltage (scaled, 0.1V resolution)
Data[7]: Status flags (bit-packed error codes)
```

#### Commands from Jetson

**Autonomous Throttle Control**:
```
ID: 0x220 (Throttle Command from Jetson)
DLC: 8 bytes
Data[0-1]: Target throttle position (0-1000, 0.1% resolution)
Data[2]: Command mode (0=manual, 1=autonomous)
Data[3-4]: Checksum or sequence counter
Data[5-7]: Reserved
```

**Engine Enable/Disable** (Safety Override):
```
ID: 0x001 (Emergency Stop)
DLC: 1 byte
Data[0]: 0x00=stop engine, 0x01=enable
```

**See** `/integration/CLAUDE.md` for complete CAN protocol specification.

---

### TunerStudio Configuration

**Software**: TunerStudio MS (Windows/Mac/Linux)
- License: Free version available; paid version recommended ($60) for full features
- Connection: Serial over USB (115200 baud) or Bluetooth (with BT module)

**Functions**:
1. **Configuration**: Set all ECU parameters (engine constants, sensor calibration)
2. **Real-Time Tuning**: Modify VE/ignition tables while engine running
3. **Autotune**: Laptop-based algorithm automatically adjusts VE table using O2 feedback
4. **Data Logging**: 10-20 samples/second, 20 parameters max per sample
5. **Gauge Displays**: Customizable dashboards for monitoring

**Configuration Workflow**:
1. Initial setup: Engine constants (displacement/2, 2 cylinders, 2-stroke, 1 injector)
2. Trigger setup: Select trigger pattern, configure VR/Hall sensor
3. Sensor calibration: CLT, IAT, TPS, O2 sensor calibration
4. Base fuel map: Load or create initial VE table (start conservative)
5. Base ignition map: Set conservative timing table (10° BTDC typical)
6. Cranking/warmup: Configure starting and warm-up enrichments
7. Idle tuning: Set idle target (e.g., 1500 RPM), adjust idle control
8. WOT tuning: Tune wide-open throttle fuel and timing
9. Part-throttle tuning: Refine cruising/part-throttle areas
10. Closed-loop (optional): Enable O2 feedback control

---

### Directory Structure (When Populated)

```
layer2-heart/
├── CLAUDE.md               # This file (domain knowledge)
├── speeduino-config/
│   ├── base_map_2stroke.msq
│   ├── tuning_log_YYYYMMDD.msl
│   └── sensor_calibrations.ini
├── firmware/
│   ├── speeduino-202501/
│   └── custom_modifications/
├── can-interface/
│   ├── obd2_telemetry.py
│   ├── autonomous_throttle.py
│   └── can_message_defs.h
├── tuning/
│   ├── ve_table_iterations/
│   ├── ignition_timing_maps/
│   └── autotune_sessions/
├── tests/
│   ├── bench_test_sensors.py
│   ├── can_loopback_test.py
│   └── injector_pulse_validation.py
└── docs/
    ├── speeduino_pinout.pdf
    ├── 2stroke_config_notes.md
    └── tuning_procedure.md
```

---

### Key References

**Root-Level Documentation**:
- [Project RoboSnomo CLAUDE.md](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md) - PM-level overview
- [Complete BOM](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md#hardware-status) - Centralized parts list

**Integration Points**:
- `/integration/CLAUDE.md` - CAN bus protocols, message IDs, timing specifications
- `/layer1-brain/CLAUDE.md` - Jetson receives ECM telemetry, sends throttle commands
- `/layer3-muscle/CLAUDE.md` - Throttle servo receives commands (mirrored from CAN or direct from ECM)

**External Resources**:
- Speeduino Wiki: https://wiki.speeduino.com/
- Speeduino GitHub: https://github.com/speeduino/speeduino
- TunerStudio: https://www.tunerstudio.com/
- Speeduino Forums: https://speeduino.com/forum/

---

### Development Priorities

**Phase 1: Procurement & Setup**
- [ ] Source Speeduino v0.4 board (STM32F407 variant, **not Arduino Mega**)
- [ ] Source MCP2551 CAN transceiver (if not on board)
- [ ] Order sensors: CLT, IAT, TPS, O2 (AEM 30-0300)
- [ ] Order high-Z fuel injector (calculate flow rate for engine displacement)

**Phase 2: Bench Testing**
- [ ] Power up Speeduino, verify voltage rails
- [ ] Connect to TunerStudio, load firmware v202501.1
- [ ] Configure engine constants (displacement/2, 2 cylinders)
- [ ] Read all sensor values, verify calibration

**Phase 3: Engine Integration**
- [ ] Install CLT, IAT, TPS sensors on engine
- [ ] Install or verify crank position sensor (Hall/VR)
- [ ] Mount fuel injector (transfer ports or intake manifold, NOT crankcase)
- [ ] Wire Speeduino ignition output to CDI trigger input

**Phase 4: Initial Tuning**
- [ ] Crank engine, verify RPM reading correct (use workaround config)
- [ ] Verify ignition timing with timing light
- [ ] Start engine on conservative VE/timing maps
- [ ] Tune idle stability (1500 RPM target)

**Phase 5: CAN Bus Integration**
- [ ] Wire MCP2551 transceiver to STM32 CAN0 pins
- [ ] Enable CAN in TunerStudio, configure 500 kbps
- [ ] Test OBD2 telemetry (candump on Jetson)
- [ ] Implement autonomous throttle control (receive 0x220 commands)

**Phase 6: Full Tuning**
- [ ] WOT tuning with wideband O2 sensor
- [ ] Part-throttle tuning for cruising
- [ ] Autotune session to refine VE table
- [ ] Ignition timing optimization (dyno or field testing)

---

### Entry/Exit Process

**When Working in this Subdirectory**:
1. Review this CLAUDE.md to understand ECM scope and 2-stroke configuration
2. Check `/integration/CLAUDE.md` for CAN bus message formats
3. Work on Speeduino configuration, tuning, or CAN interface code
4. Update this CLAUDE.md with tuning discoveries, sensor calibration notes
5. When crossing boundaries (e.g., actuator control), return to root PM level

**Returning to PM Level**:
- Navigate to `/storage/emulated/0/Documents/projects/robosnomo/`
- Read root CLAUDE.md for project-wide status
- Coordinate CAN bus integration via `/integration/`

---

**Last Updated**: November 18, 2025
**Subsystem Lead**: (TBD)
**Status**: Phase 1 - Procurement (Sourcing STM32F407-based Speeduino board)
