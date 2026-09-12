# TerraNode-ULP

**Ultra-Low-Power Environmental & Event Monitoring Node**

TerraNode-ULP is an open-source embedded platform for long-term autonomous environmental monitoring using **ESP32, low-power sensing and long-range LoRa telemetry**.

The project is designed around a simple idea:

> Build a field node that can collect useful environmental data for long periods, react to local events, and communicate its measurements over a long-range wireless link while minimizing energy consumption.

The initial target is a battery-powered outdoor node with a design goal of **12–18 months of autonomous operation**. This target will be validated through measured power consumption rather than theoretical estimates.

## Project Goals

The first version of TerraNode focuses on:

* Environmental monitoring

  * Temperature
  * Relative humidity
  * Atmospheric pressure
* Air-quality / gas sensing
* Low-power seismic or vibration event detection
* Battery voltage and power monitoring
* Long-range LoRa telemetry
* Autonomous operation using deep sleep
* Event-driven wake-up
* Modular hardware that can be expanded in future revisions

Wind sensing and other environmental sensors may be added later where they provide useful information for specific deployments.

## System Concept

The node periodically wakes from deep sleep, powers and reads the required sensors, processes the measurements locally and transmits a compact telemetry packet over LoRa.

A simplified operating cycle is:

```text
        ┌─────────────┐
        │ Deep Sleep  │
        └──────┬──────┘
               │
          RTC / Event
               │
               ▼
        ┌─────────────┐
        │ Wake & Init │
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ Sensor Read │
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ Local       │
        │ Processing  │
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ LoRa TX/RX  │
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ Return to   │
        │ Deep Sleep  │
        └─────────────┘
```

The ESP32 performs the primary sensing, control and event-processing tasks locally. A separate gateway or server can be used for long-term data collection and visualization, but the field node itself is designed to operate independently.

## Key Engineering Challenges

The project is not primarily about collecting sensor data. The main engineering challenges are:

* Achieving very low average power consumption
* Designing reliable deep-sleep and wake-up behaviour
* Managing sensor power and leakage currents
* Reliable long-range LoRa communication
* Designing a robust telemetry protocol
* Handling sensor and communication failures
* Measuring and validating real-world battery performance
* Designing hardware suitable for outdoor deployment
* Developing a modular custom PCB

## Hardware Architecture

The final hardware architecture is being developed iteratively.

The current concept consists of:

* **MCU:** ESP32
* **Radio:** LoRa, 868 MHz
* **Environmental sensing:** temperature / humidity / pressure
* **Air-quality sensing:** sensor under evaluation
* **Event sensing:** low-power accelerometer / IMU
* **Power:** Li-Ion battery with low-quiescent-current regulation
* **Battery monitoring:** voltage and current measurement
* **External sensor interfaces:** I²C / GPIO / ADC as required
* **Protection:** power filtering and transient protection
* **Mechanical:** 3D-printed outdoor enclosure

The prototype will initially use development boards and sensor modules. A custom PCB will be designed after the main architecture and component choices have been validated.

## Firmware

Firmware development is planned in **C++ using ESP-IDF / PlatformIO**.

The firmware architecture will be built around explicit operating states such as:

```text
BOOT
  ↓
WAKE REASON
  ↓
INITIALIZE
  ↓
ACQUIRE DATA
  ↓
PROCESS
  ↓
TRANSMIT
  ↓
HANDLE CONFIGURATION
  ↓
POWER DOWN
  ↓
DEEP SLEEP
```

The firmware will eventually include:

* Deep-sleep power management
* RTC-based periodic wake-up
* Hardware interrupt wake-up
* Sensor power gating
* LoRa telemetry
* Packet sequencing and validation
* Watchdog and fault recovery
* Configurable measurement intervals
* Local event detection

## Development Status

**Current stage: Architecture & early prototyping**

The initial development phase focuses on:

* Establishing LoRa communication between two ESP32/LILYGO boards
* Building the initial C++ firmware structure
* Testing environmental sensors
* Developing the telemetry format
* Measuring power consumption
* Organizing the engineering documentation
* Evaluating sensor and power-management options

The custom PCB and final enclosure will be developed after the prototype architecture has been validated.

## Roadmap

### Phase 1 — Proof of Concept

* ESP32 firmware foundation
* Sensor communication
* LoRa point-to-point communication
* Initial telemetry protocol
* Basic gateway

### Phase 2 — Low-Power Prototype

* Deep sleep
* Wake-up mechanisms
* Sensor power gating
* Battery monitoring
* Power measurements
* Event detection

### Phase 3 — System Integration

* Final sensor selection
* Telemetry protocol refinement
* Fault handling
* Outdoor communication tests
* Battery-life estimation from measured data

### Phase 4 — Custom Hardware

* KiCad schematic
* PCB layout
* Manufacturing
* PCB bring-up
* Hardware validation

### Phase 5 — Field Prototype

* 3D-printed enclosure
* Environmental protection
* Long-duration testing
* LoRa range testing
* Power and reliability characterization

## Future Development

TerraNode is intended to remain modular.

Possible future directions include:

* Solar-assisted operation
* Additional environmental sensors
* More advanced wildfire/environmental event detection
* Distributed multi-node deployments
* Autonomous deployment using UAVs
* Integration with a dedicated gateway and long-term database
* Higher-performance sensing hardware for seismic applications

These features are considered future development and are not requirements for the initial V1 prototype.

## Repository Structure

```text
TerraNode-ULP/
├── firmware/
├── hardware/
├── gateway/
├── docs/
├── measurements/
├── tests/
└── README.md
```

The repository will document both the **design process and measured results**, including hardware revisions, power measurements, communication tests and validation data.

---

**Status:** Work in progress
**Target:** Autonomous low-power field monitoring platform
**Development:** ESP32 · C++ · LoRa · KiCad · PCB Manufacturing
