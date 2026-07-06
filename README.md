<div align="center">

# 🍽️ Smart Dishwasher — Embedded Control System

### Automated Finite-State-Machine firmware for a fully embedded dishwashing controller

[![Platform](https://img.shields.io/badge/Platform-ATmega328P-orange?style=flat-square&logo=arduino)](https://www.microchip.com/en-us/product/ATmega328P)
[![Framework](https://img.shields.io/badge/Framework-Arduino-00979D?style=flat-square&logo=arduino)](https://www.arduino.cc/)
[![Language](https://img.shields.io/badge/Language-C%2B%2B-blue?style=flat-square&logo=c%2B%2B)](https://en.wikipedia.org/wiki/C%2B%2B)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](#license)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]()

</div>

---

## 📖 Overview

**Smart Dishwasher** is a complete embedded-systems project that automates the full lifecycle of a mini dishwashing machine — from water intake to drying — using a **non-blocking Finite State Machine (FSM)** running on an **ATmega328P**.

The system coordinates hydraulic, thermal, and mechanical subsystems in real time, while keeping the user informed through a **16x2 I2C LCD** and a **5-button analog keypad**, all wrapped in a safety-first design with hardware-level fault protection.

> Designed, simulated, and physically implemented end-to-end — from firmware architecture to a custom PCB.

---

## ✨ Features

| | |
|---|---|
| 🧠 **Non-blocking FSM core** | `IDLE → SELECT → FILL → WASH → DRAIN → RINSE → DRY → COMPLETE`, plus a dedicated `PAUSE` state for safety interrupts |
| 🧴 **3 wash programs** | Quick, Eco, and Heavy — each with its own duration and target temperature |
| 🔄 **Bi-directional wash motor** | Alternates clockwise / counter-clockwise every 2 seconds for full spray coverage |
| ⚡ **Active-LOW relay actuation** | Inlet valve, outlet/drain valve, dual-direction wash motors, heater, and buzzer |
| 🚪 **Real-time safety interlocks** | Opening the door mid-cycle instantly pauses the machine and de-energizes every actuator; auto-resumes when closed |
| 🌡️ **Live sensor fusion** | Water-level, digital temperature, and door-lock status — all debounced and polled non-blockingly |
| 🎨 **RGB status feedback** | Each state maps to a distinct color for instant diagnostics |
| ⌨️ **Analog resistor-ladder keypad** | 5 buttons (Right, Stop, Start, Previous, Select) with software debounce |
| 🔌 **Noise-optimized power design** | Custom PCB with a tuned LM2596 regulator to suppress motor/relay electrical noise |

---

## 🛠️ Hardware

| Component | Role |
|---|---|
| ATmega328P (Arduino) | Main controller |
| 16x2 I2C LCD | User interface / status display |
| 5-button analog keypad | Program selection & control |
| Water-level sensor | Fill-state detection |
| Digital temperature sensor | Heating control |
| Door lock switch | Safety interlock |
| Relay-driven inlet/outlet valves | Water fill & drain |
| Dual-direction wash motor relays | Spray arm rotation |
| Heating element (relay) | Water heating |
| RGB LED + buzzer | Status feedback |
| LM2596 switching regulator | Power stability / noise suppression |

---

## 🔁 State Machine Flow

```
        ┌────────┐
        │  IDLE  │ ◄────────────────────────────┐
        └───┬────┘                              │
            ▼                                   │
        ┌────────┐                              │
        │ SELECT │                              │
        └───┬────┘                              │
            ▼                                   │
        ┌────────┐   ┌────────┐   ┌────────┐    │
        │  FILL  │──►│  WASH  │──►│ DRAIN  │    │
        └────────┘   └────────┘   └───┬────┘    │
                                       ▼         │
                                  ┌────────┐     │
                                  │ RINSE  │     │
                                  └───┬────┘     │
                                      ▼          │
                                  ┌────────┐     │
                                  │  DRY   │     │
                                  └───┬────┘     │
                                      ▼          │
                                 ┌──────────┐    │
                                 │ COMPLETE │────┘
                                 └──────────┘

   ⚠ Door open at any point (except IDLE/SELECT/COMPLETE) → PAUSE
      → resumes automatically once the door is closed
```

---

## 🧪 Wash Program Parameters

| Program | Duration (approx.) | Target Temp |
|---|:---:|:---:|
| 🟢 Quick | ~56s | 50°C |
| 🔵 Eco | ~102s | 55°C |
| 🔴 Heavy | ~147s | 75°C |

---

## 🛡️ Safety Design

- **Door-open detection** triggers an immediate transition to `PAUSE`, cutting power to every actuator.
- **Emergency Stop button** forces an immediate return to `IDLE` from any state.
- **Heater fail-safe** — always forced off whenever actuators are stopped, regardless of the current state.
- **PCB layout** (replacing breadboard prototyping) with a tuned LM2596 regulator to suppress motor/relay electrical noise and prevent unexpected MCU resets under load.

---

## 📂 Firmware Structure

```
dishwasher.ino
├── Hardware & pin configuration       # ADC button thresholds, relay/sensor pins
├── RGB & actuator helpers             # setRGB_*(), stopAllActuators()
├── Keypad reader                      # Non-blocking, debounced analog read
├── Main FSM loop                      # loop() dispatches to state handlers
└── State handlers
    ├── FILL                          # Inlet valve + level sensor + timeout
    ├── WASH                          # Heater control + bi-directional motor
    ├── DRAIN                         # Outlet valve + drain motor
    ├── RINSE                         # Nested fill/wash/drain sub-phases
    ├── DRY                           # Residual-heat drying timer
    ├── COMPLETE                      # Audio-visual alert + auto-reset
    └── PAUSE                         # Safety interrupt + auto-resume
```

---

## 🚀 Getting Started

1. **Wire the hardware** according to the pin definitions at the top of `dishwasher.ino`.
2. **Install dependencies** — the `LiquidCrystal_I2C` library via the Arduino Library Manager.
3. **Flash the firmware** to an ATmega328P-based board (Arduino Uno/Nano or equivalent).
4. **Power on** — the system boots into `IDLE`, RGB LED green, LCD showing the default (Eco) program.

```bash
# Using arduino-cli
arduino-cli compile --fqbn arduino:avr:uno dishwasher.ino
arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:uno dishwasher.ino
```

---

## 🗺️ Roadmap

- [ ] Add real-time monitoring over Serial/Bluetooth
- [ ] Configurable programs via EEPROM storage
- [ ] Leak-detection sensor integration
- [ ] Mobile app / web dashboard companion

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to check the [issues page](../../issues) or open a pull request.

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

Built with ⚙️ embedded engineering and a lot of debugging patience.

</div>
