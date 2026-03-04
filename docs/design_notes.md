# Design Notes – Analog Wake Trigger

This document provides the technical background, design rationale, and internal operation of the **analog-wake-trigger** module.  
While the main README focuses on usability and integration, this file explains *how* the circuit works and *why* specific design choices were made.

---

## 1. System Overview

The analog-wake-trigger is an always‑on analog front‑end designed to detect fast variations in resistive velostat sensors.  
Its purpose is to wake the microcontroller used in the *sleep-monitor* project only when relevant motion occurs, enabling deep‑sleep operation and drastically reducing power consumption.

The circuit performs four main functions:

- High‑pass filtering of the velostat signals  
- Amplification and removal of DC components  
- Threshold detection using comparators  
- Generation of a clean, fixed‑duration digital pulse using a monostable 555  

The module consumes approximately **3 mA**, making it suitable for continuous operation.

---

## 2. Velostat Sensor Interface

The module is designed to work with the 2×2 velostat matrix used in the *sleep-monitor* project.  
Velostat behaves as a pressure‑dependent resistor, with resistance decreasing under load.

Key considerations:

- Slow variations (breathing, static weight) must be ignored  
- Only rapid changes (movement) should trigger a wake event  
- The microcontroller must be able to temporarily disconnect the module to read the sensors directly  

To achieve this, the analog-wake-trigger monitors the AC component of the sensor signals.

---

## 3. High‑Pass Filtering

A simple RC high‑pass filter removes slow variations and passes only rapid changes.

Design goals:

- Cutoff frequency low enough to ignore breathing  
- High enough to detect body movement  
- Minimal component count  

The filter outputs small AC signals centered around 0 V (after coupling).

---

## 4. Amplification Stage (NPN Transistor)

The AC signal from the high‑pass filter is small and must be amplified before comparison.

The NPN amplifier provides:

- Gain to bring the signal into a usable range  
- Removal of the DC component  
- Low output impedance for the comparator stage  

Five BC337 transistors are used to process the four sensor lines and combine their activity.

---

## 5. Threshold Detection (LM393 Comparator)

Two LM393 comparators detect when the amplified signal exceeds a user‑defined threshold.

Reasons for choosing LM393:

- Open‑collector output  
- Low power consumption  
- Good noise immunity  
- Operates at 3.3 V and 5 V  

A potentiometer allows the user to adjust sensitivity depending on:

- Mattress thickness  
- Sensor placement  
- User weight  
- Desired trigger level  

Hysteresis can be added if needed to reduce noise‑induced false triggers.

---

## 6. Pulse Generation (NE555 Monostable)

Movement events are often short and irregular.  
To ensure the microcontroller reliably wakes, the comparator output triggers a **monostable 555**.

The 555 generates:

- A clean digital pulse  
- Fixed duration (~1 second, adjustable)  
- Output independent of input pulse width  

This ensures:

- The microcontroller always receives a valid wake signal  
- The system remains robust even with noisy or brief motion events  

---

## 7. Microcontroller Integration

The module connects to a GPIO pin configured as a wake‑from‑sleep interrupt.

Workflow:

1. Module monitors sensors continuously (~3 mA).  
2. Movement → comparator → 555 → wake pulse.  
3. Microcontroller wakes.  
4. Microcontroller disables the module (high‑impedance mode).  
5. Microcontroller reads the velostat matrix directly.  
6. Microcontroller re‑enables the module and returns to sleep.  

This architecture allows the *sleep-monitor* to operate in an event‑driven manner rather than continuous polling.

---

## 8. LTspice Simulation

The full circuit was simulated in LTspice before breadboard testing.

Simulations include:

- High‑pass filter response  
- Amplifier gain and bandwidth  
- Comparator switching behavior  
- 555 monostable timing  
- Noise sensitivity  

Waveforms and `.asc` files are included in the `/hardware/ltspice/` directory.

---

## 9. Breadboard Validation

The circuit was assembled on a breadboard to validate:

- Real‑world sensitivity  
- Noise performance  
- Threshold adjustment range  
- Pulse duration stability  
- Interaction with the *sleep-monitor* sensors  

Measured current consumption: **~3 mA**.

---

## 10. Future Improvements

Possible enhancements:

- Add hysteresis to comparator stage  
- Replace 555 with a low‑power CMOS timer  
- Add a latch to ensure pulse capture  
- Create a PCB version  
- Add LED indicators for debugging  

---

