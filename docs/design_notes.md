# Design Notes

This document explains the design rationale and internal operation of the **analog‑wake‑trigger** module.  
While the main README focuses on usage and integration, this file describes *how* the circuit works and *why* specific design choice was made.

---

## System Overview

### The circuit performs five main functions:
1. High‑pass filtering of the velostat sensor signals  
2. Amplification of filtered signals
3. Threshold cross detection using comparators  
4. Generation of a clean, fixed‑duration digital pulse using a monostable NE555  
5. Switching the sensors into a high‑impedance state so the MCU can read them without interference from the module  

Additionally, the circuit must generate several internal voltage references, described in:

6. Voltage References

---

## 1. High‑Pass Filtering

### Design goals
- Cutoff frequency high enough to ignore breathing  
- Low enough to detect body movement  
- Minimal component count  

### Design notes
- The base capacitor together with the BJT amplifier forms a high‑pass filter with a simulated cutoff frequency of about 1.2 Hz, which worked well with the real circuit.
- The collector signal must then be centered at Vref/2 so it sits halfway between Vt+ and Vt− (the comparator thresholds).
- This is achieved using a 10 µF coupling capacitor and a 470 kΩ–470 kΩ divider, which form a second high‑pass stage.
- This second stage must have a cutoff frequency much lower than 1.2 Hz to preserve a first‑order response.
- Its cutoff frequency is Fc = 1/(2pi*235kΩ * 10uF) = 0.07 Hz.

---

## 2. Amplification Stage (NPN Transistor)

### Design goals
- A BJT amplifier was chosen instead of an op‑amp because it is simpler and avoids the need for a dual‑supply rail, allowing direct power from the MCU.
- A virtual ground using an RC divider could emulate an op‑amp midpoint, but at such low frequencies the required capacitor would be very large. The BJT amplifier avoids this issue.

### Design notes
- The purpose of the amplification is to increase the separation between Vt+ and Vt− (see section 6. Voltage References), so small mismatches between the Vref divider (10 kΩ – 10 kΩ potentiometer – 10 kΩ) and the HPF reference divider (470 kΩ – 470 kΩ) are not critical.

---

## 3. Comparators and Diode OR‑Gate

### Design goals
- The LM393 was chosen because it is widely available and operates from a 3 V supply, making it compatible with both 3.3 V and 5 V MCUs.
- Its open‑collector outputs make it ideal for the diode‑OR logic stage that follows.

### Design notes
- The resting input signal sits at Vref/2.
- The midpoint between Vt+ and Vt− is also Vref/2 (ignoring resistor mismatch).
- Whenever the signal exceeds Vt+ or drops below Vt−, the corresponding comparator pulls its output LOW (~0 V).
- Any LOW output forward‑biases its diode and pulls Vtrig LOW (~0.7 V), effectively implementing an OR logic gate.

---

## 4. Pulse Generation (NE555 Monostable)

Movement events are often short and irregular. To ensure the microcontroller reliably wakes, the comparator output triggers a **monostable NE555**.

The 555 provides:
- A clean digital pulse  
- Fixed duration (adjustable; with 470 kΩ and 1 µF, T = 1.1RC ≈ 0.5 s)  
- Output independent of input pulse width  

This guarantees reliable MCU wake‑up even with noisy or brief motion events.

---

## 5. High‑Impedance Sensor Switching

- The sensors are powered through a PNP transistor on Vcc and an NPN transistor on GND.
- When these transistors are ON, the sensors are powered and form voltage dividers.
- When they are OFF, the sensors are effectively disconnected from the supply and become high‑impedance.
- Because the PNP is active‑LOW and the NPN is active‑HIGH, an additional NPN transistor is used as an inverter so the MCU’s EN signal can switch both devices simultaneously.

---

## 6. Voltage References

### Vref
- The LM393 only accepts input voltages between 0 V and Vcc − 2 V.  
  Vref ensures that all comparison thresholds remain within this valid range.
- The BJT base is biased at [18/(10+18)] * Vcc = 0.64*Vcc
- With a small base current, Vbe ~= 0.5V, so Vref = 0.64Vcc - 0.5
- Vref(Vcc=3.3) = 1.6V
- Vref(Vcc=5)  =  2.7V
  

### Vt+ and Vt−
- These are the comparator thresholds. When the amplified signal rises above Vt+ or falls below Vt−, the corresponding comparator output goes LOW.
- With the 10 kΩ – 10 kΩ potentiometer – 10 kΩ divider, Vt+ can reach at most two‑thirds of Vref, ensuring it stays below Vcc − 2 V for Vcc = 3.3 V (or higher).

---

