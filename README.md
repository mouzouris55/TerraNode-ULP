# TerraNode-ULP

**Ultra-Low-Power Environmental & Event Monitoring Node**

TerraNode-ULP is an embedded outdoor monitoring platform built around a low-power MCU, LoRa telemetry, environmental sensing and event-driven firmware.

The V1 design is intentionally focused on a small number of useful measurements rather than maximizing the sensor count. The project is being developed as a complete engineering system: requirements, component selection, prototype measurements, firmware, custom PCB, mechanical integration and field validation.

> **Design principle:** measured behaviour and documented test results take priority over theoretical or marketing claims.

## V1 Objective

The first TerraNode version is a battery-powered outdoor field node with a design target of long autonomous operation.

The current design target is approximately **12–18 months of operation from a ~6000–6500 mAh 1S2P 18650 battery pack**. This is a design target, not a validated runtime claim. The actual energy budget will be derived from measured sleep, sensor, processing and LoRa current profiles.

V1 focuses on:

- Temperature, relative humidity and atmospheric pressure
- Low-power vibration/seismic event detection
- Local optical smoke/particle event sensing
- Wind speed
- Wind direction
- Rainfall
- Battery voltage/current monitoring
- LoRa 868 MHz telemetry
- Event-driven wake-up and low-power operation
- Outdoor mechanical integration
- Measurement-based power and communication validation

## System Architecture

The current architecture is:

```text
                    ┌─────────────────────────────┐
                    │        TerraNode V1         │
                    │                             │
                    │        ESP32-C6 MCU         │
                    │              │              │
                    │      ┌───────┴────────┐     │
                    │      │                │     │
                    │    Sensors          LoRa    │
                    │      │              868MHz  │
                    │      │                │     │
                    │      └───────┬────────┘     │
                    │              │              │
                    │       Power Management      │
                    │              │              │
                    │        1S2P Li-Ion          │
                    └─────────────────────────────┘
```

The main processing MCU and LoRa radio are separate RF domains:

- **LoRa:** 868 MHz
- **ESP32-C6 native RF:** 2.4 GHz, used only if Wi-Fi/BLE functionality is required

The V1 does not depend on the ESP32-C6's 2.4 GHz radio for field telemetry; LoRa is the primary long-range link.

## Sensor Set — V1

### BME280

Measures:

- Temperature
- Relative humidity
- Atmospheric pressure

Interface: I²C.

The sensor is sampled periodically and does not need to remain continuously active.

### LIS3DH

The LIS3DH is the current V1 low-power accelerometer.

Primary functions:

- Wake-on-motion / threshold detection
- Vibration event detection
- Low-power acceleration monitoring
- Event-triggered data capture

The LIS3DH is **not currently claimed as a scientific seismograph**. Reliable seismic magnitude estimation would require substantially more validation, including sensor noise characterization, mechanical coupling, sampling strategy, calibration and signal analysis.

During development, alternative accelerometers may be compared, for example:

- ADXL362 for low-power comparison
- Higher-bandwidth vibration accelerometers as development/reference instruments

The decision will be based on measured noise, bandwidth, current consumption and event-detection performance rather than component price alone.

### GP2Y1010AU0F

The V1 uses an optical particle/smoke sensor instead of a continuously powered MOX gas sensor.

The reason is the power budget. MOX sensors such as SGP30/BME680 require heater operation and are less attractive for a long-autonomy ULP node.

The GP2Y1010AU0F is intended for:

- Local smoke/particle event detection
- Particle-event logging
- Experimental correlation with environmental measurements

It is **not** treated as a universal fire detector or toxicity detector.

Distinguishing events such as smoke versus agricultural spray is considered a future classification problem using multiple measurements and temporal behaviour.

### Wind Speed

The anemometer provides pulses, typically through a reed switch.

The node should not rely on a short instantaneous sample. Pulses need to be accumulated while the main processing system is sleeping.

The ESP32-C6 LP domain is therefore an important part of the architecture. ESP-IDF provides an LP-core GPIO pulse-counter example specifically demonstrating pulse counting while the main CPU is in deep sleep. citeturn0search5

The firmware can use the accumulated count to calculate average wind behaviour over a measurement interval and may also record a gust-related metric.

### Wind Direction

The wind vane provides an analogue voltage through a resistor network.

The divider will be power-gated and measured only when required, reducing continuous leakage.

The voltage-to-direction mapping will be calibrated for the actual vane used.

### Rain Gauge

The rain gauge produces pulses from a tipping-bucket/reed-switch mechanism.

The pulse counter must continue operating while the main CPU sleeps so that tips are not missed between scheduled wake-ups.

At the end of a measurement interval the main CPU reads the accumulated count, converts it to rainfall and resets/updates the counter.

## Low-Power Architecture

The basic firmware cycle is:

```text
WAKE
  ↓
CHECK WAKE REASON
  ↓
INITIALIZE REQUIRED HARDWARE
  ↓
READ ENVIRONMENTAL SENSORS
  ↓
READ WIND / RAIN COUNTERS
  ↓
PROCESS / VALIDATE DATA
  ↓
BUILD TELEMETRY PACKET
  ↓
LORA TX
  ↓
OPTIONAL RX WINDOW
  ↓
STORE REQUIRED STATE
  ↓
POWER DOWN
  ↓
SLEEP
```

The ESP32-C6 LP domain can remain responsible for low-power background tasks such as pulse counting while the main CPU is sleeping.

The exact sleep mode, GPIO mapping, wake sources and timing will be validated on hardware.

## Power Management

The V1 power architecture is based on minimizing both active consumption and leakage.

Candidate/current concepts include:

- 1S2P 18650 Li-Ion battery
- Low-quiescent-current LDO
- Switched sensor power rails
- MOSFET-gated voltage dividers
- Battery voltage monitoring
- Local decoupling and bulk capacitance
- Appropriate protection and filtering

Voltage dividers are treated as potential continuous loads and will be power-gated where useful.

The theoretical average-current limits for a 6500 mAh battery are approximately:

- 12 months: ~0.74 mA average
- 18 months: ~0.49 mA average

These are theoretical limits, not usable design targets. The real budget must include leakage, regulator consumption, sensor duty cycle, LoRa transmissions, battery losses, temperature, aging and self-discharge.

## Power Measurement

The **Nordic Power Profiler Kit II (PPK2)** is the primary development tool for power profiling.

It supports external custom hardware and can measure from the sub-µA range to 1 A, with high-speed current profiling up to 100 kS/s.

The following profiles will be measured separately:

- Deep sleep
- LP-domain operation
- Wake-up
- BME280 measurement
- GP2Y1010AU0F measurement pulse
- LIS3DH monitoring
- Wind/rain pulse monitoring
- LoRa TX
- LoRa RX
- Complete measurement/transmission cycle
- Sensor retry/fault cases

The goal is to calculate actual charge per cycle and average current rather than estimating battery life from individual datasheet numbers.

## Seismic / Vibration Noise Strategy

Noise is treated as both an electrical/sensor problem and a mechanical/environmental problem.

### Electronic/sensor noise

The development measurements will examine:

- Sensor noise floor
- Noise density
- RMS noise
- Peak-to-peak noise
- Sampling behaviour
- Bandwidth
- Power consumption

### Mechanical/environmental noise

Potential sources include:

- Wind
- Rain
- Human movement
- Nearby machinery
- Enclosure resonance
- Cable movement
- Mechanical mounting
- Ground/environmental vibration

Testing will progress from:

1. Sensor on the bench
2. Sensor mounted on prototype
3. Sensor mounted in the enclosure
4. Final mechanical mounting
5. Outdoor measurements
6. Controlled vibration tests

Raw data can be analysed using RMS, FFT/PSD and event statistics.

FFT does not need to run continuously on the field node. It can be performed during development or in the gateway/backend.

## Smoke / Environmental Event Philosophy

The node should primarily measure and report data.

More complex interpretation belongs in the gateway/backend:

- Event classification
- Fire confidence
- Smoke versus spray classification
- Multi-node correlation
- Advanced seismic analysis
- Long-term anomaly detection

The node should not make unsupported claims from a single sensor.

## LoRa Telemetry

The field link uses **LoRa at 868 MHz**.

The existing LILYGO LoRa32 boards can be used for the initial point-to-point prototype.

The final PCB may use an ESP32-C6 module plus a suitable external LoRa transceiver/module.

The following parameters will be selected and validated experimentally:

- Spreading factor
- Bandwidth
- Coding rate
- TX power
- Packet interval
- Antenna configuration

No fixed range claim will be made until real range and packet-loss measurements exist.

## Telemetry Packet

The packet is intended to remain compact and versioned.

Possible fields include:

- Protocol version
- Node ID
- Sequence number
- Timestamp/time delta
- Temperature
- Humidity
- Pressure
- Particle/smoke measurement
- Wind speed
- Wind direction
- Rainfall
- Battery voltage
- Event flags
- Error/status flags
- Seismic event summary
- CRC

Sequence numbers allow packet loss to be identified. Protocol versioning allows future firmware changes without silently breaking the receiver.

## Firmware Architecture

Firmware development is planned in C++ using ESP-IDF/PlatformIO.

The state-machine structure will follow concepts such as:

```text
BOOT
WAKE_REASON
INIT_POWER
INIT_SENSORS
READ_ENVIRONMENT
READ_WIND_RAIN_COUNTERS
CHECK_SEISMIC_EVENT
PROCESS_DATA
BUILD_PACKET
LORA_TX
LORA_RX
STORE_STATE
FAULT_RECOVERY
SLEEP
```

Important firmware features:

- Deep sleep
- RTC-based wake-up
- LP-domain background tasks
- Hardware interrupt wake-up
- Sensor power gating
- LoRa telemetry
- Packet sequencing
- CRC/status validation
- Watchdog
- Sensor timeout/retry
- Fault recovery
- Configurable measurement interval

A failed sensor should not normally stop the entire node.

## Mechanical Architecture

The system is divided into two physical functions.

### Central enclosure

Contains:

- Main PCB
- ESP32-C6
- LoRa interface
- Battery
- Power management
- Seismic/vibration sensing interface

The seismic sensor requires a mechanically stable mounting arrangement.

### External sensor pod

Contains the environmental/smoke sensing hardware and should provide:

- Airflow
- Solar/radiation protection
- Rain protection
- Serviceability

A Stevenson-screen-inspired 3D-printed structure can be used for atmospheric sensing.

The atmospheric sensor placement and seismic sensor placement are intentionally different because their mechanical requirements are different.

## PCB Design Philosophy

The PCB will be designed to be compact, practical and easy to validate.

The design priorities are:

- Short current loops
- Local decoupling
- Short connections between ICs and their support components
- Solid and low-impedance ground
- Clear power domains
- Separation of noisy/high-current areas from sensitive measurements
- Clean RF section
- Test points for validation
- Easy probing during bring-up

### Important distinction: digital routing vs RF routing

The board does **not** require blanket signal-length matching for ordinary low-speed I²C/GPIO/SPI connections.

The critical exception is the RF path.

For the ESP32-C6 RF section, Espressif specifies controlled **50 Ω RF impedance**, an appropriate matching network close to the RF device and a properly designed antenna interface. RF routing should be short, consistent and isolated from other high-frequency signals. citeturn0search0turn0search1

For an external 868 MHz LoRa module, the exact RF layout must follow that module/transceiver's reference design.

Therefore:

- No unnecessary length matching on normal sensor buses
- No unnecessary serpentine traces
- Keep digital connections short and clean
- Keep RF routing as a dedicated transmission-line problem
- Maintain the required RF impedance
- Respect antenna keep-out
- Keep noisy clocks/UART/USB away from the antenna
- Place RF matching components according to the selected RF reference design

The board can be made compact, but the antenna and RF section must not be compressed blindly just to reduce PCB area.

## Current-Loop / Noise Strategy

Small current loops are particularly important around:

- LDO input/output capacitors
- Any switching regulator
- LoRa supply path
- ESP32 supply decoupling
- high-current transient paths

The goal is to minimize loop area and parasitic inductance where current changes quickly.

At the same time, "put everything as close together as possible" is not a universal rule.

Examples:

- BME280 should be kept away from heat sources such as the ESP32/regulator where practical.
- Smoke/particle sensing needs suitable airflow and mechanical placement.
- The LoRa antenna needs physical clearance.
- The seismic sensor needs mechanically appropriate mounting.
- Sensitive analogue paths should not be routed through noisy power/RF regions.

Compactness is a goal, but correct physical separation is sometimes more important.

## Prototype Hardware

Existing development hardware will be used wherever possible.

Primary prototype resources include:

- 2× LILYGO TTGO LoRa32 V2.1_1.6
- ESP32 development boards
- BME280 module
- LIS3DH module
- existing sensor/component kits
- prototype PCBs/breadboards
- logic analyzer
- oscilloscope
- PPK2
- laboratory power supply when required

## Development Sequence

### Phase 1 — Architecture

- Freeze V1 sensor set
- Define power domains
- Define interfaces
- Define telemetry packet
- Define wake/sleep strategy
- Review datasheets

### Phase 2 — Bench Proof of Concept

- ESP32-C6 firmware foundation
- LoRa point-to-point link
- BME280
- LIS3DH
- GP2Y1010AU0F
- Wind/rain pulse counting
- Wind direction ADC
- Basic battery measurement

### Phase 3 — Measurement

- PPK2 power profiling
- Sensor current comparison
- Seismic noise characterization
- LoRa range/packet-loss tests
- Fault/recovery tests

### Phase 4 — Schematic and PCB

- KiCad schematic
- Component selection
- Datasheet review
- Power tree
- RF section
- PCB layout
- DRC
- BOM/CPL/Gerbers

### Phase 5 — PCB Bring-up

- Power rails
- Programming/debug
- MCU
- Sensors
- LoRa
- Test points
- Current profiling

### Phase 6 — Mechanical Integration

- 3D-printed enclosure
- Sensor pod
- Seismic mounting
- Cable glands/connectors
- Environmental protection

### Phase 7 — Field Validation

- Long-duration power measurements
- LoRa range testing
- Outdoor sensor testing
- Weather/noise characterization
- Reliability testing

## Engineering Validation

Every important claim should eventually have evidence.

Examples:

| Claim | Required evidence |
|---|---|
| Low power | PPK2 current profile |
| 12–18 month target | Measured average current + battery model |
| LoRa range | Range/packet-loss test |
| Seismic event detection | Controlled vibration + outdoor noise data |
| Low sensor noise | Raw acceleration/noise measurements |
| Outdoor enclosure | Environmental/mechanical testing |
| Reliable sensor recovery | Fault injection/recovery tests |

## What V1 Does Not Claim

Until validated, the project will not claim:

- 12–18 months of guaranteed battery life
- Certified IP68 protection
- Scientific-grade seismograph performance
- Accurate earthquake magnitude estimation
- Universal fire detection
- Toxic-gas detection from the particle sensor
- Guaranteed long-range LoRa performance
- Zero-noise operation

The documentation will use terms such as **design target**, **experimental prototype**, **measured**, **validated**, **event detector** and **IP68-targeted** according to the actual evidence.

## Future Development

Possible V2/future directions:

- Solar-assisted autonomous operation
- Advanced seismic signal processing
- Multi-node seismic network
- Time synchronization and localization
- Additional environmental sensing
- VOC/MOX sensing in a higher-power platform
- GNSS
- Local data storage
- Expanded weather-station functions
- Autonomous UAV deployment
- TerraNode as a UAV-deployed sensor payload

These are not V1 requirements.

## Repository Structure

```text
TerraNode-ULP/
├── README.md
├── LICENSE
├── .gitignore
├── hardware/
│   ├── architecture/
│   ├── schematics/
│   ├── pcb/
│   ├── bom/
│   └── datasheets/
├── firmware/
│   ├── node/
│   └── gateway/
├── docs/
│   ├── development_log.md
│   ├── design_decisions.md
│   ├── testing.md
│   ├── deployment.md
│   └── media/
├── measurements/
│   ├── power/
│   ├── lora/
│   └── sensors/
└── tests/
    ├── hardware/
    ├── firmware/
    └── system/
```

## Development Status

**Current stage: Architecture definition and early prototype preparation**

The immediate goal is to have a coherent technical architecture and preliminary component selection before implementation begins.

The next engineering step is detailed datasheet review for each selected component, followed by prototype measurements and power profiling.

---

**Status:** Work in progress  
**Platform:** ESP32-C6 · LoRa 868 MHz · C++ · ESP-IDF · KiCad  
**Focus:** Ultra-low-power embedded sensing, telemetry and field validation
