# Analog Wake Trigger – Low‑Power Motion Detector for the *sleep-monitor* Project

A compact analog module that monitors the velostat pressure sensors from the **sleep-monitor** project and generates a digital wake signal whenever movement is detected.  
It was designed specifically to integrate with the *sleep-monitor* system  
(https://github.com/antonio-mariano/sleep-monitor), but it can also be used in similar frameworks or as a standalone movement detector.

---

## 📌 Overview

In the original *sleep-monitor*, the microcontroller must stay awake to continuously read the sensors, which leads to high power consumption.  
The **analog-wake-trigger** solves this limitation by acting as a low‑power motion sentinel that consumes only **~3 mA** while continuously monitoring the sensors.

This allows the Raspberry Pi Pico W (or any compatible microcontroller) to remain in deep‑sleep mode and wake only when relevant motion occurs.

High‑level operation:

- The module continuously monitors the velostat sensors.
- When movement is detected, it generates a pulse of approximately 1 second (configurable).
- This pulse wakes the microcontroller, which performs a full sensor reading (as in the original *sleep-monitor*) and then returns to sleep.
- The microcontroller can temporarily place the velostat sensors in high impedance from the module so they can be read without interference.

**BLOCK_DIAGRAM**

A detailed explanation of the internal circuit is available in:  
👉 `docs/design_notes.md`

---

## ✨ Features

- **Compatible with both 3.3 V and 5 V microcontrollers**, allowing direct power from the MCU.
- **Low power consumption (~3 mA)**, suitable for battery‑powered systems.
- **Generates a clean digital pulse** to wake a microcontroller from deep sleep.
- **Adjustable detection threshold** via potentiometer for sensitivity tuning.
- **Designed as a companion module for the sleep-monitor project.**
- **Can also operate as a standalone movement detector**, where the pulse simply indicates “movement detected”.

---

## 🧩 Components

- Velostat pressure sensor matrix from the *sleep-monitor* project (or any resistive sensors arranged in a 2×2 matrix).
- 5× BC337 (NPN) transistors  
- 1× BC327 (PNP) transistor  
- 2× LM393 comparators  
- 1× NE555 timer in monostable configuration  
- Resistors, capacitors, and a potentiometer  

---

## 🔌 Circuit Assembly

The module is composed of several analog stages that process the velostat signals and generate a clean digital wake pulse.  
These stages include filtering, amplification, threshold detection, and monostable pulse generation.

A complete schematic (KiCad + PDF) is available in the `/hardware` directory.  
LTspice simulations and waveforms are available in `/hardware/ltspice
