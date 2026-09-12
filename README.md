# 🌍 TerraNode - ULP Environmental & Seismic LoRa Node

## 📌 Project Overview
An open-source, Ultra-Low Power (ULP) autonomous sensor node built on the ESP32 architecture. Designed for long-term field deployment (12-18 months), this project combines atmospheric telemetry with edge-computed seismic detection. 

The system leverages LoRa RF communication to transmit critical data while maintaining extreme energy efficiency through dynamic Deep Sleep scheduling and hardware-level interrupts.

## 🚀 Core Architecture & Features
* **Dynamic Sleep Intervals:** The node adjusts its deep sleep cycles (e.g., 5 to 30 minutes) dynamically. Following each transmission, the node opens a brief RX window to receive configuration payloads (like RTC sync or interval updates) from the central gateway.
* **Edge Computing (FFT):** Seismic and vibration data are processed locally on the ESP32. By applying Fast Fourier Transform (FFT) algorithms, the node distinguishes genuine tectonic events from anthropogenic noise (wind, passing vehicles).
* **Event-Driven Wakeups:** The microcontroller remains in deep sleep to conserve power. However, if a high-sensitivity IMU detects acceleration exceeding a defined threshold, a hardware interrupt immediately wakes the ESP32 to evaluate the event.
* **Battery & Thermals:** Designed for maximum stability with minimal parasitic drain, utilizing switched voltage dividers for battery monitoring and ULP LDOs.

## 📡 Sensor Array
### Base Implementation
* **T/H/P:** Barometric pressure, humidity, and temperature monitoring (BME280).
* **Air Quality:** CO2 and VOC toxicity tracking.
* **Seismic Activity:** Vibration module evaluating event duration, peak intensity, and calculating a seismic probability factor.

### Future Expandability (Reserved I/O)
* **Ambient Light / UV:** Illuminance tracking for solar efficiency evaluation.
* **Mechanical Weather Sensors:** Rain gauge and anemometer integration using zero-power reed switch interrupts.

## 🗺️ Development Roadmap

**Phase 1: Sensor Prototyping & C++ Fundamentals**
* Bring up ESP32 with PlatformIO.
* Develop basic I2C communication for environmental sensors.
* Implement raw accelerometer data reading, threshold triggers, and basic FFT signal processing.

**Phase 2: RF Communication Link**
* Establish Point-to-Point (P2P) LoRa communication between two nodes (e.g., LILYGO TTGO LoRa32).
* Define the byte-struct payload for optimized, low-bandwidth transmission.
* Implement the post-TX RX window for bidirectional communication.

**Phase 3: Full Integration & Power Optimization (Test Bench)**
* Combine all sensor libraries and LoRa transmissions on a single test bench.
* Implement deep sleep routines and hardware interrupt wake-ups.
* Finalize the payload structure and verify dynamic interval updates.

**Phase 4: Custom Hardware & Enclosure Deployment**
* **PCB Design:** Route a custom board featuring the ESP32-WROOM, an external LoRa SMD module (868 MHz), a switched voltage divider, and an MCP1700 LDO for power management.
* **Mechanical Design:** Design and 3D print a weather-proof ASA enclosure. Incorporate external SMA bulkhead connectors for the antenna and O-ring sealing for environmental protection.

## 🔧 Hardware Stack (Target)
* **MCU:** ESP32 (LILYGO TTGO LoRa32 for prototyping -> ESP32-WROOM for final PCB)
* **RF:** LoRa 868 MHz (ISM band for EU compliance)
* **Power:** 18650 Li-Ion Cell, ULP Voltage Regulation, optional trickle-charge Solar Panel
