# 🚦 IoT-Based Smart Traffic Management System

<div align="center">

[![Arduino](https://img.shields.io/badge/Platform-Arduino_UNO-00979D?style=for-the-badge&logo=arduino&logoColor=white)](https://www.arduino.cc/)
[![C++](https://img.shields.io/badge/Language-C%2FC%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)](https://isocpp.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![IoT](https://img.shields.io/badge/Domain-IoT%20%26%20Smart%20Cities-brightgreen?style=for-the-badge)](#)
[![Build Status](https://img.shields.io/badge/Build-Passing-success?style=for-the-badge)](#)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-blue?style=for-the-badge)](CONTRIBUTING.md)

**A real-time, sensor-driven traffic signal controller built on Arduino UNO that dynamically adapts signal timing based on vehicle density — with priority override for emergency vehicles.**

[Features](#-features) • [Architecture](#-system-architecture) • [Hardware](#-hardware-setup) • [Getting Started](#-getting-started) • [Roadmap](#-roadmap) • [Contributing](#-contributing)

</div>

---

## 📌 Problem Statement

Urban intersections worldwide suffer from **fixed-cycle traffic signals** that ignore real-time congestion. This results in:
- 🚗 Unnecessary idling and fuel waste at empty red lights
- 🚑 Delayed emergency vehicle response times
- 📈 Compounding congestion during peak hours

This project proposes a **sensor-driven adaptive signal controller** as a foundational prototype for smart city infrastructure.

---

## ✨ Features

| Feature | Description | Status |
|---|---|---|
| 🔍 Real-Time Density Detection | LDR/IR sensors measure per-lane vehicle density | ✅ Implemented |
| ⏱ Adaptive Signal Timing | Green time scales proportionally to detected density | ✅ Implemented |
| 🚨 Emergency Override | Servo motor opens dedicated emergency lane on trigger | ✅ Implemented |
| 💡 LED Signal Simulation | Full R/Y/G signal per lane on breadboard | ✅ Implemented |
| 📊 Serial Monitor Logging | Real-time state output at 9600 baud | ✅ Implemented |
| 📡 Wireless Data Logging | ESP8266 Wi-Fi module integration | 🔄 In Progress |
| 🤖 AI Adaptive Control | ML-based signal prediction | 📋 Planned |
| 📱 Mobile Dashboard | Remote monitoring & emergency override app | 📋 Planned |

---

## 🏗 System Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    SMART TRAFFIC CONTROLLER                     │
│                                                                 │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  SENSORS │───▶│  ARDUINO UNO │───▶│    OUTPUT DEVICES    │  │
│  │          │    │              │    │                      │  │
│  │ LDR × 4  │    │  ATmega328P  │    │  LED Signals × 4     │  │
│  │ IR  × 4  │    │  16 MHz      │    │  Servo Motor × 1     │  │
│  │ TRIG BTN │    │  32KB Flash  │    │  Serial Monitor      │  │
│  └──────────┘    └──────────────┘    └──────────────────────┘  │
│                         │                                       │
│                  ┌──────▼──────┐                                │
│                  │  LOGIC CORE │                                │
│                  │             │                                │
│                  │ • Density   │                                │
│                  │   Scoring   │                                │
│                  │ • Priority  │                                │
│                  │   Queue     │                                │
│                  │ • Timer     │                                │
│                  │   Control   │                                │
│                  └─────────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

### Signal State Machine
```
         ┌──────────────────────────────────────────┐
         │                                          │
    ┌────▼─────┐    density     ┌──────────────┐   │
    │  ASSESS  │─────scored────▶│  SELECT LANE │   │
    │  DENSITY │                │  (highest)   │   │
    └──────────┘                └──────┬───────┘   │
                                       │           │
                              ┌────────▼────────┐  │
                              │   GREEN PHASE   │  │
                              │  (3s – 10s      │  │
                              │   adaptive)     │  │
                              └────────┬────────┘  │
                                       │           │
                              ┌────────▼────────┐  │
                              │  YELLOW PHASE   │  │
                              │    (2s fixed)   │  │
                              └────────┬────────┘  │
                                       │           │
                              ┌────────▼────────┐  │
                              │   RED PHASE     │  │
                              │  (all others)   │──┘
                              └─────────────────┘

         ⚡ EMERGENCY INTERRUPT overrides any state instantly
```

---

## 🔌 Hardware Setup

### Bill of Materials

| Component | Qty | Purpose | Approx. Cost |
|---|---|---|---|
| Arduino UNO R3 | 1 | Microcontroller | $10–$25 |
| LDR (Light Dependent Resistor) | 4 | Vehicle density sensing | $0.10 ea |
| IR Sensor Module (optional) | 4 | Enhanced detection | $1.50 ea |
| Red LED | 4 | Stop signal | $0.05 ea |
| Yellow LED | 4 | Caution signal | $0.05 ea |
| Green LED | 4 | Go signal | $0.05 ea |
| SG90 Servo Motor | 1 | Emergency lane gate | $2–$5 |
| Tactile Push Button | 1 | Emergency trigger | $0.10 |
| 220Ω Resistors | 12 | LED current limiting | $0.05 ea |
| 10kΩ Resistors | 4 | LDR voltage dividers | $0.05 ea |
| Breadboard (830 tie) | 1 | Prototyping | $5 |
| Jumper Wires (M-M) | 40 | Connections | $3 |
| USB Cable (Type-B) | 1 | Programming/Power | $3 |
| **Total** | | | **~$35–$60** |

### Pin Mapping
```
ARDUINO UNO PIN ASSIGNMENTS
═══════════════════════════════════════════════

DIGITAL OUTPUT — LED Signals
  Lane 1 (North):  Red=2,  Yellow=3,  Green=4
  Lane 2 (East):   Red=5,  Yellow=6,  Green=7
  Lane 3 (South):  Red=8,  Yellow=9,  Green=10
  Lane 4 (West):   Red=11, Yellow=12, Green=13

DIGITAL OUTPUT — Servo Motor
  Emergency Gate:  Pin=A5 (PWM via Servo lib)

ANALOG INPUT — LDR Sensors
  Lane 1: A0   Lane 2: A1   Lane 3: A2   Lane 4: A3

DIGITAL INPUT — Emergency Button
  Emergency Trigger: Pin=A4 (INPUT_PULLUP)
```

### Wiring Schematic
```
                    ARDUINO UNO
                  ┌────────────┐
     LDR1 ────── A0            2 ── 220Ω ── RED_LED_1
     LDR2 ────── A1            3 ── 220Ω ── YLW_LED_1
     LDR3 ────── A2            4 ── 220Ω ── GRN_LED_1
     LDR4 ────── A3            5 ── 220Ω ── RED_LED_2
  EMG_BTN ────── A4           ...
  SRV_SIG ────── A5           13 ── 220Ω ── GRN_LED_4
                  │
        GND ─────┘   5V ──┬── LDR pulldowns (10kΩ to GND)
                           └── Servo VCC
```

---

## 🚀 Getting Started

### Prerequisites

- [Arduino IDE 2.x](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- Arduino UNO driver installed
- `Servo.h` library (bundled with Arduino IDE)

### Installation
```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/smart-traffic-system.git
cd smart-traffic-system

# 2. Open in Arduino IDE
# File → Open → src/smart_traffic_main/smart_traffic_main.ino

# 3. Select Board & Port
# Tools → Board → Arduino UNO
# Tools → Port → COMx (Windows) or /dev/ttyUSB0 (Linux/Mac)

# 4. Upload
# Click Upload (→) or press Ctrl+U
```

### Configuration

Edit `src/config.h` to tune system behavior:
```cpp
// Timing parameters (milliseconds)
#define GREEN_MIN_MS       3000   // Minimum green phase
#define GREEN_MAX_MS      10000   // Maximum green phase
#define YELLOW_MS          2000   // Fixed yellow phase
#define DENSITY_SAMPLE_MS   500   // Sensor polling interval

// Density thresholds (0–1023 ADC range)
#define DENSITY_LOW        300    // Sparse traffic
#define DENSITY_MEDIUM     600    // Moderate traffic
#define DENSITY_HIGH       800    // Dense traffic

// Emergency
#define EMERGENCY_DURATION_MS  15000  // Emergency lane open time
```

---

## 💻 Core Logic Overview

### Adaptive Timing Algorithm
```cpp
// Green time is proportional to density score vs total density
int computeGreenTime(int laneDensity, int totalDensity) {
    if (totalDensity == 0) return GREEN_MIN_MS;
    float ratio = (float)laneDensity / totalDensity;
    int adaptiveTime = GREEN_MIN_MS + (int)(ratio * (GREEN_MAX_MS - GREEN_MIN_MS));
    return constrain(adaptiveTime, GREEN_MIN_MS, GREEN_MAX_MS);
}
```

### Priority Queue Logic
```
Each cycle:
  1. Read all 4 lane sensor values
  2. Normalize to density score (0–100)
  3. Sort lanes by density (descending)
  4. Grant green to highest-density lane
  5. Compute green duration proportionally
  6. Advance to next highest → repeat
```

---

## 📁 Project Structure
```
smart-traffic-system/
│
├── 📂 src/                          # All source code
│   ├── smart_traffic_main/
│   │   └── smart_traffic_main.ino   # Main Arduino sketch
│   ├── config.h                     # Tunable parameters
│   ├── TrafficController.h          # Controller class header
│   ├── TrafficController.cpp        # Controller logic
│   ├── SensorManager.h              # Sensor abstraction
│   └── EmergencyHandler.h           # Emergency override logic
│
├── 📂 hardware/                     # Hardware documentation
│   ├── schematic.pdf                # Full wiring schematic
│   ├── BOM.csv                      # Bill of materials
│   └── pin_mapping.md               # Detailed pin reference
│
├── 📂 simulations/                  # Software simulations
│   ├── traffic_sim.py               # Python traffic flow simulator
│   └── signal_timing_test.py        # Timing validation script
│
├── 📂 tests/                        # Test cases
│   ├── test_density_scoring.cpp     # Unit tests
│   ├── test_timing_algorithm.cpp
│   └── test_emergency_override.cpp
│
├── 📂 docs/                         # Documentation
│   ├── SYSTEM_DESIGN.md             # Architecture deep-dive
│   ├── CALIBRATION.md               # Sensor calibration guide
│   └── FUTURE_ENHANCEMENTS.md       # Roadmap details
│
├── 📂 .github/
│   ├── workflows/ci.yml             # GitHub Actions CI
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
│
├── README.md                        # This file
├── CONTRIBUTING.md                  # Contribution guidelines
├── LICENSE                          # MIT License
└── CHANGELOG.md                     # Version history
```

---

## 🗺 Roadmap

### v1.0 — Current (Prototype) ✅
- Sensor-based density detection
- Adaptive green timing
- Emergency vehicle override
- Serial logging

### v2.0 — In Development 🔄
- [ ] ESP8266 Wi-Fi integration for cloud data logging
- [ ] MQTT protocol for IoT dashboard (ThingSpeak / MQTT broker)
- [ ] Persistent traffic log to SD card
- [ ] Multi-intersection coordination via I2C

### v3.0 — Planned 📋
- [ ] TensorFlow Lite model for predictive signal control
- [ ] Camera-based vehicle counting (ESP32-CAM)
- [ ] Flutter mobile app for remote monitoring & override
- [ ] PCB design (transition from breadboard to custom PCB)

---

## 🧪 Testing

### Functional Tests Performed

| Test Case | Expected | Result |
|---|---|---|
| All LEDs light in sequence | R→Y→G per lane | ✅ Pass |
| Idle lane gets minimum green | 3s green | ✅ Pass |
| Dense lane gets max green | 10s green | ✅ Pass |
| Emergency overrides active green | Immediate RED + servo open | ✅ Pass |
| System resumes after emergency | Normal cycle | ✅ Pass |

### Running Python Simulations
```bash
cd simulations/
pip install -r requirements.txt
python traffic_sim.py --lanes 4 --cycles 20 --scenario peak_hour
```

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first.
```bash
# Fork → Clone → Branch → Code → Test → PR
git checkout -b feature/your-feature-name
git commit -m "feat: add your feature"
git push origin feature/your-feature-name
```

**Good first issues:** sensor calibration improvements, simulation scenarios, documentation.

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---


  
