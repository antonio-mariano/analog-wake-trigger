# Analog Wake Trigger – Low‑Power Motion Detector for the *sleep-monitor* Project

A compact analog module that monitors the velostat (resistive) pressure sensors from the **sleep-monitor** project and generates a digital wake signal whenever movement is detected.
It was designed specifically to integrate with the **sleep-monitor** project:  https://github.com/antonio-mariano/sleep-monitor



---

## 📌 Overview

In the original *sleep-monitor*, the microcontroller must stay awake to continuously read the sensors, resulting in high power consumption. This new module solves that limitation by acting as a low‑power motion sentinel.
The module consumes only **~3 mA**, allowing to put the Raspberry Pi Pico W (or the microcontroller in use) in deep sleep mode and wake only when relevant motion occurs.


The operating principle is simple:

- The module continuously monitors the sensors.
- When movement occurs, it produces a pulse of approximately 1 second (configurable duration).
- This pulse wakes the microcontroller, which performs a full sensor reading (like in the original *sleep-monitor*) and then returns to sleep
- The microcontroller can temporarily disable the module, effectively placing the velostat sensors in high impedance so they can be read without interference

BLOCK_DIAGRAM 

---

## ✨ Features
- **Compatible with both 3.3 V and 5 V microcontrollers.**, allowing direct power from the microcontroller
- **Low power consumption (~3 mA)**, suitable for battery‑powered systems
- **Generates a  digital pulse** to wake a microcontroller from deep sleep
- **Adjustable detection threshold** via potentiometer, enabling sensitivity tuning
- **Designed as a companion module for the sleep-monitor project.**
- **Can also operate as a standalone movement detector**, where the wake‑up pulse simply indicates “movement detected” rather than triggering a detailed reading.

---

## 🧩 Components
- Velostat pressure sensor matrix from the *sleep-monitor* project (or any resistive sensors arranged in a 2×2 matrix).
- 5× BC337 (NPN) transistors.
- 1× BC327 (PNP) transistor.
- 2× LM393 comparators — detect when movement exceeds the threshold.
- 1× NE555 timer in monostable configuration — generates the wake‑up pulse.
- Resistors, capacitors, and a potentiometer.

---

## 🔌 Circuit Assembly

## How it Works
The signal path is composed of simple, well‑defined analog stages:

1. **Sensor → High‑Pass Filter**  
   Removes slow variations such as breathing or static weight.

2. **High‑Pass → NPN Amplifier**  
   Boosts small AC variations and removes the DC component.

3. **Amplifier → LM393 Comparator**  
   Compares the signal against a user‑adjustable threshold to detect motion.

4. **Comparator → 555 Monostable**  
   Produces a clean, fixed‑duration digital pulse (~1 s), independent of the input shape.

5. **555 Output → Microcontroller Wake Pin**  
   Wakes the Raspberry Pi Pico W or any other MCU configured for wake‑from‑sleep.

The circuit was first validated in LTspice and will be assembled on a breadboard for real‑world testing.

---

## 🚀 How to Use
- Connect the module output (555 output) to a GPIO pin configured as a wake‑from‑sleep interrupt.  
- Put the microcontroller into deep sleep.  
- When movement occurs, the module outputs a ~1 s high pulse.  
- Make the microcontroller wake, disconnect the module from the sensors, then read the sensors (as in the *sleep-monitor* project), and return to sleep.  
- Adjust the comparator threshold to match the original *sleep-monitor* characteristics

  
