# Analog Wake Trigger – Low‑Power Motion Detector for the *sleep-monitor* Project

A compact analog module that monitors the velostat pressure sensors used in the **sleep-monitor** project and generates a clean digital wake signal whenever movement is detected.  
Although designed specifically for integration with [github.com/antonio-mariano/sleep-monitor](https://github.com/antonio-mariano/sleep-monitor), the module can also be used in similar systems or as a standalone movement detector.

This README focuses on circuit assembly, usage, and integration.  
A detailed explanation of the circuit architecture, design rationale, and the analog techniques behind it is available at:
👉 [docs/design_notes.md](https://github.com/antonio-mariano/analog-wake-trigger/blob/main/docs/design_notes.md)

---

## 📌 Overview

In the original *[sleep-monitor](https://github.com/antonio-mariano/sleep-monitor)*, the microcontroller must remain awake to continuously read the sensors, which results in high power consumption.  
The **analog-wake-trigger** solves this limitation by acting as a low‑power motion sentinel that consumes only **3 mA** while continuously monitoring the sensors.

This allows the microcontroller (MCU) to stay in deep‑sleep mode and wake only when relevant motion occurs.

High‑level operation:

- 1 — The module continuously monitors the velostat sensors.  
- 2 — When movement is detected, it generates a half‑second pulse (configurable).  
- 3 — This pulse wakes the microcontroller, which performs a full sensor reading (as in the original *sleep-monitor*) and then returns to sleep.  
- The microcontroller can temporarily place the velostat sensors in high impedance so they can be read without interference from the module.

![Overview](images/overview.gif)

---

## ✨ Features

- Compatible with both **3.3 V and 5 V** microcontrollers (can be powered directly from the MCU).  
- **Low power consumption** (~3 mA), ideal for battery‑powered systems.  
- **Generates a clean digital pulse** to wake a microcontroller from deep sleep.  
- **Adjustable detection threshold** via potentiometer for sensitivity tuning.  
- Designed as a **companion module** for the sleep-monitor project.  
- Can also operate as a **standalone movement detector**, where the output pulse simply indicates “movement detected”.

---

## 🧩 Components (BOM)

The circuit was designed to minimize component count and use common, easy‑to‑source parts.

- Velostat pressure sensor matrix from the [*sleep-monitor*](https://github.com/antonio-mariano/sleep-monitor) project (or any resistive sensors arranged in a 2×2 matrix)  
- 5× BC337 (NPN) transistors  
- 1× BC327 (PNP) transistor  
- 2× LM393 comparators  
- 1× NE555 timer  

- 2× 1.5 kΩ resistors  
- 8× 10 kΩ resistors  
- 4× 18 kΩ resistors  
- 4× 450 kΩ resistors  
- 2× 1.2 MΩ resistors  
- 1× 10 kΩ potentiometer  

- 1× 0.01 µF capacitor  
- 3× 1 µF capacitors  
- 2× 10 µF capacitors  

- 4× 1N4148 diodes  

---

## 🔌 Circuit Assembly

![Circuit Schematic KiCad](images/schematic.png)

The module is composed of several analog stages that process the velostat signals and generate a clean digital wake pulse.  
These stages include filtering, amplification, threshold‑crossing detection, and monostable pulse generation.

A complete schematic (KiCad + PDF) is available in the `/hardware` directory.  
LTspice simulations and waveforms are available in `/hardware/ltspice`.

---
