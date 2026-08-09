# SirenSense V2

**Offline acoustic intelligence and driver-assistance platform**

[Website](https://sirensense.es) · [Early firmware prototype](https://github.com/ibrra3/SirenSense) · [Project video](https://www.youtube.com/watch?v=NpQYkkEUuDs)

SirenSense helps drivers notice, identify and locate approaching emergency vehicles when traffic, cabin noise or reflections between buildings make a siren difficult to interpret. The core system processes audio locally, classifies the acoustic event and presents a directional warning without depending on cloud inference.

The project received **first prize at Hack4Change 2026** at the ETSII, Universidad de Sevilla.

## Why this project exists

Hearing a siren does not always reveal where it is coming from. Reflections in narrow streets can create a misleading direction, while music, insulation or hearing loss can delay detection altogether. SirenSense approaches the problem as a complete embedded system: microphone timing, signal processing, edge inference, power integrity, wireless communication and the driver interface all have to work together.

## System architecture

```text
Four-microphone roof array
  ESP32-S3 + Edge AI + GCC-PHAT/TDOA
                 |
              ESP-NOW
                 |
Cabin controller             Emergency-vehicle transponder
  ESP32-S3 <---- LoRa ---->  ESP32-S3 + GPS
       |
  UART JSON
       |
Raspberry Pi Zero 2 W HUD
  alerts, GPS, offline map, diagnostics and camera modules
```

### Roof acoustic node

One ESP32-S3 captures four INMP441 microphones through two synchronized I2S controllers. Each I2S data line carries two microphones by assigning one microphone to the left slot and the other to the right slot. The two buses share clock timing through the ESP32-S3 GPIO matrix, use deterministic startup ordering and run a per-boot alignment and health check.

The processing pipeline combines an Edge Impulse Conv1D classifier with GCC-PHAT time-difference-of-arrival estimation. Spectral zero-padding and parabolic refinement provide sub-sample delay estimates, while a coherence gate suppresses bearings that are not trustworthy.

### Cabin controller and HUD

The cabin unit combines an ESP32-S3 controller with a Raspberry Pi Zero 2 W display system. The microcontroller owns the real-time peripherals and forwards normalized telemetry to the Pi using newline-delimited JSON over UART. The Pi renders the driver interface and can also run independently in simulation mode for development and testing.

## Product functions

The project is larger than siren detection alone. The current prototype and its validated software modules cover:

1. Offline siren detection and acoustic-event classification
2. 360-degree direction estimation from a four-microphone array
3. Priority driver alerts for acoustic detections and RF beacons
4. LoRa emergency-vehicle beacon and transponder architecture
5. GPS speed, course and position display
6. NAJM offline mapping with track-up and north-up modes
7. Close-range radar alert integration
8. Driver-fatigue monitoring and priority warning overlays
9. Dashcam, event gallery and incident-logging modules
10. Environmental and system telemetry, including air-quality and temperature inputs
11. Voice alerts, microSD event logging and diagnostics
12. Experimental V16-style emergency beacon mode

Some modules are still under validation. SirenSense is an engineering prototype, not a certified automotive, V16 or emergency-services product.

## Hardware engineering

The electronics were designed in KiCad and include custom main and microphone PCBs. The roof unit uses a LiFePO4 power system with battery protection, charging, DC conversion and a separately filtered microphone rail. Decoupling capacitors and ferrite beads were added close to the digital microphones to reduce conducted noise and stabilize audio capture.

Key components include:

* ESP32-S3-WROOM-1-N16R8 modules
* Four INMP441 digital MEMS microphones
* Raspberry Pi Zero 2 W cabin computer
* SX1278 LoRa radio and GPS modules
* LiFePO4 battery, BMS, TP5000 charger and MT3608 conversion stage
* Custom KiCad PCBs and microphone interconnect boards

## Engineering status

| Area | Current state |
| --- | --- |
| Offline siren classification | Implemented and demonstrated |
| Four-channel audio acquisition | Implemented with two I2S stereo buses |
| Direction-finding pipeline | Implemented; final vehicle-axis field calibration remains |
| Custom electronics | Designed, fabricated and assembled for the prototype |
| Pi HUD and offline map | Working software prototype with simulation support |
| Camera integration | Software implemented; hardware capture reliability still being investigated |
| LoRa security | Prototype architecture; production key provisioning is not finalized |
| Automotive certification | Not started; this is not a production safety device |

## Repository scope

This repository is the public engineering case study for V2. The project spans several hardware nodes and software environments, and the source is being consolidated into clean, reproducible modules before broader release. The earlier ESP32 firmware iteration remains available in the [SirenSense repository](https://github.com/ibrra3/SirenSense), while the product overview and demonstrations are published at [sirensense.es](https://sirensense.es).

## Technical stack

`C` · `C++` · `Python` · `ESP32-S3` · `FreeRTOS` · `PlatformIO` · `ESP-IDF` · `Edge Impulse` · `GCC-PHAT` · `I2S` · `ESP-NOW` · `LoRa` · `UART` · `Raspberry Pi` · `KiCad`

## Author

Designed and built by **Ibrahim Najjar**, with a focus on embedded systems, real-time firmware, edge AI and electronics.

