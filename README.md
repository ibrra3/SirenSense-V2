# SirenSense: Urban Ambulance Acoustic Detection System

SirenSense is an open-source, Edge-AI acoustic monitoring node designed for Smart City intersections. Its primary goal is to autonomously detect the presence of approaching ambulances in heavy traffic and transmit preemptive alerts to municipal traffic light controllers, reducing emergency response times and preventing intersection collisions.

## 🚀 Overview

Unlike vision-based systems that fail in blind spots, occlusions, or poor weather conditions, SirenSense relies purely on acoustic signatures. By leveraging TinyML on low-cost microcontrollers, the system continuously listens to the urban soundscape, classifies specific ambulance siren patterns (like wail, yelp, and two-tone), and broadcasts hardware-level alerts via Long Range (LoRa) radio.

This repository contains the firmware and machine learning deployment code required to build a standalone intersection monitoring node.

## 🛠️ Hardware Architecture

The hardware is designed to be mounted on traffic poles and is built around a low-power, high-efficiency edge processing unit:

- **Core Processor:** ESP32-S3-N16R8 (Dual-core, optimized for digital signal processing and AI workloads).
- **Acoustic Sensors:** INMP441 Digital I2S MEMS Microphones for high SNR audio capture.
- **Telemetry & Comms:** LoRa SX1278 RA-02 (433MHz) transceiver for long-range, low-power transmission to traffic infrastructure.
- **Diagnostics Display:** 2.42" OLED display via I2C/SPI for on-site maintenance, calibration, and debugging.
- **Environmental Monitoring:** DS18B20 1-Wire temperature sensor to monitor outdoor enclosure conditions and prevent overheating.

## 🧠 Software & AI Pipeline

- **Digital Signal Processing (DSP):** The system samples I2S audio and computes Mel-Filterbank Energy (MFE) spectrograms. This isolates the critical 600Hz-1500Hz frequency range typical of ambulance sirens, effectively filtering out standard urban background noise like wind, engines, and voices.
- **TinyML Classification:** A Convolutional Neural Network (CNN) runs directly on the ESP32-S3. It processes the spectrograms locally and offline, classifying the audio into "Siren" or "Noise" with high confidence.
- **Privacy by Design:** To ensure public privacy and bandwidth efficiency, no raw audio is ever recorded or uploaded. Only the binary classification trigger and a timestamp are transmitted over the LoRa network.

## ⚙️ Getting Started

### Prerequisites

- ESP-IDF v5.0+ or PlatformIO
- C++14 compatible compiler
- Edge Impulse Inferencing Library (for the ESP32)

### Installation & Wiring

1. Clone the repository:
    
    ```
    git clone https://github.com/yourusername/SirenSense-Urban.git
    ```
    
2. Wire the INMP441 microphones to the ESP32-S3 I2S pins. Ensure the L/R channel selection is properly grounded or pulled to 3.3V depending on your schematic.
3. Connect the LoRa SX1278 module via SPI.
4. Build and flash the firmware to your ESP32-S3 development board.
5. Monitor the serial output at `115200` baud rate to view the real-time AI confidence scores and DSP extraction times.


---

```
# SirenSense-Urban: Edge-AI Acoustic Node for Smart Intersections 🚑🚦

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Platform](https://img.shields.io/badge/platform-ESP--IDF%20v5.0-blue)
![Hardware](https://img.shields.io/badge/hardware-ESP32--S3-orange)
![License](https://img.shields.io/badge/license-MIT-green)

**SirenSense-Urban** is an open-source, privacy-preserving acoustic monitoring node designed to preemptively detect approaching emergency vehicles (ambulances, fire trucks, police) in dense urban environments.

By deploying these low-cost nodes on traffic light poles, municipalities can interface directly with traffic controllers via Long Range (LoRa) radio to switch lights to green *before* the emergency vehicle enters the intersection, drastically reducing response times and preventing fatal collisions.

---

## 📖 Table of Contents
- [Why Acoustic Detection?](#why-acoustic-detection)
- [System Architecture](#system-architecture)
- [Hardware Bill of Materials (BOM)](#hardware-bill-of-materials-bom)
- [Software & RTOS Pipeline](#software--rtos-pipeline)
- [Wiring & Schematic](#wiring--schematic)
- [Installation & Flashing](#installation--flashing)
- [LoRa Payload Structure](#lora-payload-structure)
- [Contributing](#contributing)

---

## 🎯 Why Acoustic Detection?
Traditional emergency preemption systems rely on optical strobes (which fail around blind corners or in heavy fog) or cloud-connected GPS (which suffers from urban canyon multipath delays).

SirenSense relies purely on **acoustic signatures**. Sound waves diffract around buildings, allowing the system to "hear" an ambulance approaching blocks away, even when completely occluded from the intersection's line of sight. To ensure **100% citizen privacy**, no raw audio is ever recorded, saved, or transmitted. All microphone data is processed locally on the edge via TinyML, and only a binary "Siren Detected" trigger is transmitted over the air.

---

## 🧠 System Architecture

The node operates on a strictly constrained power and memory budget, utilizing a dual-core architecture to prevent audio sampling bottlenecks.

1. **Audio Acquisition:** A stereo pair of digital MEMS microphones captures ambient street noise via an I2S bus. Direct Memory Access (DMA) ring buffers are used to prevent CPU blocking.
2. **Digital Signal Processing (DSP):** Raw PCM audio is windowed and transformed into Mel-Filterbank Energy (MFE) spectrograms, isolating the 600 Hz – 1500 Hz frequency bands where siren sweeps (wail, yelp, hi-lo) reside.
3. **Machine Learning Inference:** A lightweight Convolutional Neural Network (CNN) analyzes the spectrograms in real-time to classify the acoustic event, rejecting background noise like heavy wind, car horns, and construction.
4. **Wireless Telemetry:** Upon a positive classification with >85% confidence, an encrypted interrupt packet is broadcasted via a 433MHz LoRa transceiver to the intersection's main traffic controller.

---

## 🛠️ Hardware Bill of Materials (BOM)

| Component | Description | Protocol/Logic |
| :--- | :--- | :--- |
| **ESP32-S3-WROOM-1-N16R8** | Dual-core Tensilica LX7 MCU | 3.3V Logic |
| **INMP441 (x2)** | Digital Bottom-Ported MEMS Microphones | I2S |
| **SX1278 RA-02** | Long Range (LoRa) Transceiver Module | SPI (433MHz) |
| **SSD1306 2.42"** | Diagnostic OLED Display | I2C |
| **DS18B20** | Weatherproof Temperature Sensor | 1-Wire |

*Note: The hardware is designed to be housed in a 3D-printed, acoustically transparent weatherproof dome.*

---

## ⚙️ Software & RTOS Pipeline
The firmware is built using **FreeRTOS** on the ESP-IDF. To maintain sub-100ms latency, tasks are strictly pinned to specific cores:

* **Core 0 (Control & I/O):**
  * `task_i2s_dma_read`: Manages continuous double-buffered audio sampling.
  * `task_lora_tx`: Handles radio state machine and MAC layer transmission.
  * `task_sensors`: Polls the DS18B20 to monitor internal enclosure temperatures.
* **Core 1 (Math & ML Workloads):**
  * `task_dsp_compute`: Executes STFT and Mel-spectrogram generation.
  * `task_edge_impulse_infer`: Runs the quantized TensorFlow Lite Micro (TFLite) CNN model.

---

## 🔌 Wiring & Schematic

### I2S Microphone Array (Stereo Configuration)
To achieve synchronized stereo sampling for future Time Difference of Arrival (TDOA) updates, both microphones share the same clock lines, but the `L/R` channels are pulled to opposite logic levels.

| INMP441 Pin | ESP32-S3 Pin | Notes |
| :--- | :--- | :--- |
| **VDD** | 3.3V | Require 100nF decoupling capacitor |
| **GND** | GND | Star-ground configuration |
| **SCK (BCLK)** | GPIO 41 | Shared between both mics |
| **WS (LRCLK)**| GPIO 42 | Shared between both mics |
| **SD (DATA)** | GPIO 2 | Shared data line |
| **L/R (Mic 1)** | GND | Sets Mic 1 to Left Channel |
| **L/R (Mic 2)** | 3.3V | Sets Mic 2 to Right Channel |

### LoRa SX1278 (SPI)
| SX1278 Pin | ESP32-S3 Pin |
| :--- | :--- |
| **MOSI** | GPIO 11 |
| **MISO** | GPIO 13 |
| **SCK** | GPIO 12 |
| **NSS (CS)** | GPIO 10 |
| **DIO0 (IRQ)** | GPIO 9 |

---

## 🚀 Installation & Flashing

### Prerequisites
* Ensure you have [ESP-IDF v5.0+](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/get-started/) installed and configured.
* Python 3.8+ for deployment scripts.

### Build Instructions
1. Clone the repository and update submodules:
   ```bash
   git clone https://github.com/YourUsername/SirenSense-Urban.git
   cd SirenSense-Urban
   git submodule update --init --recursive
```

2. Set the target to ESP32-S3:
    
    ```
    idf.py set-target esp32s3
    ```
    
3. Configure the environment variables (LoRa frequency, mic gain):
    
    ```
    idf.py menuconfig
    # Navigate to "SirenSense Node Configuration"
    ```
    
4. Build, flash, and monitor the serial output:
    
    ```
    idf.py build flash monitor
    ```
    

---

## 📡 LoRa Payload Structure

To minimize airtime and comply with ISM band duty-cycle regulations, the alert payload is heavily compressed into an 8-byte hexadecimal string.

**Byte Layout:** `[Node_ID (2B)] [Event_Type (1B)] [Confidence (1B)] [Temp (1B)] [Battery_mV (2B)] [Checksum (1B)]`

**Example Payload:** `0x4A1C 0x02 0x5F 0x1A 0x0CE4 0x8B`

- `0x4A1C` -> Node ID: 18972 (Intersection 5th & Main)
- `0x02` -> Event: Yelp Siren Detected
- `0x5F` -> ML Confidence: 95%
- `0x1A` -> Enclosure Temp: 26°C
- `0x0CE4` -> Battery: 3300 mV (3.3V)
- `0x8B` -> CRC8 Checksum

---

## 🤝 Contributing

We welcome pull requests from acoustic engineers, embedded developers, and urban planners. If you are submitting DSP optimizations or new TFLite models, please ensure your changes do not exceed the 100ms processing latency constraint.

See `CONTRIBUTING.md` for our code of conduct and branching guidelines.

## 🚧 Future Roadmap

- **Direction of Arrival (DoA):** Implementing a Time Difference of Arrival (TDOA) algorithm to calculate exactly which street the ambulance is approaching from.
- **Solar Integration:** Adding a deep-sleep polling routine and a TP5000 charging circuit for completely off-grid solar-powered operation.

## 🛡️ License

Distributed under the MIT License. See `LICENSE` for more information.
