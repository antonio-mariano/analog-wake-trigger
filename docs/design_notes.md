# Design Notes

This document provides the design rationale and internal operation of the **analog-wake-trigger** module.  
While the main README focuses on usability and integration, this file explains *how* the circuit works and *why* specific design choices were made.

---

## System Overview

### The circuit performs five main functions:
- 1 High‑pass filtering of the velostat sensor signals  
- 2 Amplification
- 3 Threshold detection using comparators  
- 4 Generation of a clean, fixed‑duration digital pulse using a monostable 555
- 5 Put the sensors in high impedance, since the MCU must be able to read the sensors without interference from the module

Additionally, it has to generate some voltage references for operation, exmplained in
- 6 Voltage References
---

## 1. High-Pass Filtering
### Design goals:
- Cutoff frequency high enough to ignore breathing  
- Low enough to detect body movement  
- Minimal component count

### Design notes:
- The base capacitor with the BJT amplifier forms a filter, with a (LTSpice simulation) cutoff frequency of about 1.2 Hz, wich worked well in the real circuit
- The colector signal needs then to be centered at Vref/2 to be in the middle point between Vt+ and Vt- (threshold to the comparators)
- This is done with the decoupling 10uF capacitor with the 470k-470k divider, that forms another filter
- This 2nd stage filter needs to have cutoff frequency << 1.2 Hz, because we want to keep a first order behaviour 
- The cuttoff frequency of that 2nd stage is Fc = 1/(2pi*235k * 10uF) = 0.07 Hz
- In practie it only moves the DC value to Vref/2, keeping the desired 1st order behaviour

---

## 2. Amplification Stage (NPN Transistor)
### Design goals:
- A BJT amplifier was chosen instead of OpAmp because it's simpler and avoids the use of dual-supply source, allowing direct power from the MCU
- A RC voltage divider could be used to simulate a middle point OpAmp ground, but since we are dealing with such low frequencies, the C would be huge, so the BJT amplifer avoids that

### Design notes:
- The purpose of the amplification is to increase space between Vt+ and Vt- (view Voltage References section), so any resistor mismatch between the Vref generation (10k - Potentiometer - 10k) and the DC reference for the HPF (470k - 470k) isn't critic

---

## 3. Comparators and Diode OR-Gate
### Design goals:
- The LM393 was chosen becouse it is wildly available, and operates from 3V supply, allowing to be supplied from 3.3V and 5V MCUs/boards.
- Also it is open-collector, making it ideal for the diodes OR logic-gate that comes next.

### Design notes:
- The resting input signal is at Vref/2
- the middle point betweeh Vt+ and Vt- is always Vref/2 (ignoring resistor mismatch)
- Whenever the signal overcomes Vt+ or Vt-, the respective comparator puts its output in LOW state (~0V)
- Any comparator output in LOW state makes the respective diode conduct and puts Vtrig at LOW (~0.7V), therefore acting like an OR logic gate

---

## 4. Pulse Generation (NE555 Monostable)

Movement events are often short and irregular. To ensure the microcontroller reliably wakes, the comparator output triggers a **monostable NE555**.

The 555 generates:

- A clean digital pulse  
- Fixed duration (adjustable: T = 1.1 * 470k * 1uF ~= 0.5s)  
- Output independent of input pulse width  

This ensures the system remains robust even with noisy or brief motion events

## 5. High Impedance circuit

- The sensors are powered throw a PNP transistor at Vcc and a NPN at GND.
- When these transistors are ON, the sensors are powered ON, creating voltage dividers.
- When these transistors are OFF, the sensors are effectively disconnected from the power supply
- Since the PNP base is active at LOW, and the NPN base activates at high, an aditional NPN is used as an inverter, so the EN signal (controlled by the MCU) can activate both transistors at the same time


## 6. Voltage References

### Vref
- The LM393 comparator only accepts inputs that go from 0 to Vcc-2 V. The purpose of Vref is to generate a voltage reference that makes comparation voltages below Vcc-2V.
- The base of the BJT is at Vcc*18/(10+18) = 0.64*Vcc, and for very small current Vbe_on = 0.5 V, so Vref = 0.64Vcc - 0.5

### Vt+ and Vt-
- These are the comparation (threshold) voltages. When the amplified signal is above Vtresh+ or below Vt-, the respecive comparator output goes low.
- Because of the 10k voltage divider with 10k potentiometer, Vt+ is at maximum 2/3 of Vref, so below Vcc-2 for Vcc = 3.3V (or above)





