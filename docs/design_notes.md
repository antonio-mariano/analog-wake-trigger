# Design Notes

This document provides the technical background, design rationale, and internal operation of the **analog-wake-trigger** module.  
While the main README focuses on usability and integration, this file explains *how* the circuit works and *why* specific design choices were made.

---

## System Overview

### The circuit performs four main functions:
- High‑pass filtering of the velostat sensor signals  
- Amplification
- Threshold detection using comparators  
- Generation of a clean, fixed‑duration digital pulse using a monostable 555
- Put the sensors in high impedance, since the MCU must be able to read the sensors without interference from the module

---


## 2. High-Pass Filtering
### Design goals:
- Cutoff frequency high enough to ignore breathing  
- Low enough to detect body movement  
- Minimal component count

### Design notes:
- The base capacitor with the BJT amplifier forms a filter, with a simulated (LTSpice) cutoff frequency of about 1.2 Hz
- The colector signal needs then to be centered at Vref/2 for the comparators
- This is done with the decoupling 10uF capacitor with the 470k-470k divider, that forms another filter
- This 2nd stage filter needs to have cutoff frequency << 1.2 Hz, because we want to keep a first order behaviour 
- The cuttoff frequency of that 2nd stage is w = 2pi*f = 1/(RC) where R = 470k//470k = 235k and C = 10uF => fc = 0.07 Hz, so in practie it only moves the DC value to Vref/2

## 1. Amplification Stage (NPN Transistor)
### Design goals:
- A BJT amplifier was chosen instead of OpAmp because it's simpler and avoids the use of dual-supply source, allowing direct power from the MCU
- Note that a RC voltage divider could be used to simulate a middle point OpAmp ground, but since we are dealing with such low frequencies, the C would be huge, so the BJT amplifer avoids that

### Design notes:
- The purpose of the amplification is to increase space between Vt+ and Vt- (view Voltage References section), so any resistor mismatch between the Vref generation (10k - Potentiometer - 10k) and the DC reference for the HPF (470k - 470k) isn't critic


---

## 3. Comparators and Diode OR-Gate

Two LM393 comparators detect when the amplified signal exceeds a user‑defined threshold.

Reasons for choosing LM393:
- Availability
- Open‑collector output  
- Low power consumption   
- Operates at 3.3 V and 5 V


- The resting input signal is at Vref/2
- the middle point betweeh Vt+ and Vt- is always Vref/2 (ignoring resistor mismatch)
- Whenever the signal overcomes Vt+ or Vt-, the respective comparator puts its output in LOW state (~0V)
- Any comparator output in LOW state makes the respective diode conduct and puts Vtrig at LOW (~0.7V), therefore acting like an OR logic gate

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

  
- The signals at the comparator input can oscilate and trigger the comparator multiple times in sequence. The purpose of the pulse generator is to give to the MCU a clean unique Pulse signal
- The pulse duration is given by T = 1.1RC. With the chosen values, T = 1.1 * 470k * 1uF ~= 0.5s


---



## 7. Microcontroller Integration

The module connects to a GPIO pin configured as a wake‑from‑sleep interrupt.

Workflow:

1. Module monitors sensors continuously
2. Movement → comparator → 555 → wake pulse.  
3. Microcontroller wakes.  
4. Microcontroller disables the module (high‑impedance mode).  
5. Microcontroller reads the velostat matrix directly.  
6. Microcontroller re‑enables the module and returns to sleep.  

This architecture allows the *sleep-monitor* to operate in an event‑driven manner rather than continuous polling.

---

---

---

## 8. LTspice Simulation

The full circuit was simulated in LTspice before breadboard testing.

Simulations include:

- High‑pass filter response  
- Amplifier gain and bandwidth  
- Comparator switching behavior  


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

## High Impedance circuit

- The sensors are powered throw a PNP transistor at Vcc and a NPN at GND.
- When these transistors are ON, the sensors are powered ON, creating voltage dividers.
- When these transistors are OFF, the sensors are effectively disconnected from the power supply
- Since the PNP base is active at LOW, and the NPN base activates at high, an aditional NPN is used as an inverter, so the EN signal (controlled by the MCU) can activate both transistors

## 2. Voltage References

### Vref
- The LM393 comparator only accepts inputs that go from 0 to Vcc-2 V. The purpose of Vref is to generate a voltage reference that makes comparation voltages below Vcc-2V.
- The base of the BJT is at Vcc*18/(10+18) = 0.64*Vcc, and for very small current Vbe_on = 0.5 V, so Vref = 0.64Vcc - 0.5

### Vt+ and Vt-
- These are the comparation (threshold) voltages. When the amplified signal is above Vtresh+ or below Vt-, the respecive comparator output goes low.
- Because of the 10k voltage divider with 10k potentiometer, Vt+ is at maximum 2/3 of Vref, so below Vcc-2 for Vcc = 3.3V (or above)

