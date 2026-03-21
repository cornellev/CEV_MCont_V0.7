# Bring-Up Guide

This document describes the recommended procedure for safely bringing up the DRV8353FH motor controller for the first time. Motor controllers can fail catastrophically if powered incorrectly, so the goal of this guide is to verify each subsystem step-by-step before applying full voltage or connecting a motor.

Always use a **current-limited bench supply** during initial testing.

---

# Equipment Required

Recommended tools:

- bench power supply with current limiting
- multimeter
- oscilloscope
- low-voltage test power source (12–24 V recommended)

Optional but helpful:

- current probe
- differential probe for phase measurements

---

# Step 1 — Visual Inspection

Before applying power, carefully inspect the board.

Check for:

- incorrect MOSFET orientation
- solder bridges on fine-pitch ICs
- incorrect capacitor polarity
- damaged components
- loose connectors

Visually confirm that the DRV8353, MOSFETs, and regulators are correctly oriented.

---

# Step 2 — Continuity Checks

Use a multimeter to check for unintended shorts.

Important measurements:

**DC bus to ground**

Measure resistance between:

VIN (48–60 V bus)

and

GND

This should not read as a short circuit.


**Gate to source**

Verify that MOSFET gates are not shorted to source.


**Shunt resistors**

Verify that the current sense resistors measure approximately their expected values.

---

# Step 3 — Low Voltage Power-Up

Apply a **low voltage supply** to the bus input.

Recommended initial voltage:

12–24 V

Set the power supply current limit to a conservative value such as:

0.5–1 A

Observe the supply current immediately after power is applied.

If the current limit triggers or the current rises unexpectedly, disconnect power immediately and inspect the board.

---

# Step 4 — Verify Power Rails

With the board powered at low voltage, verify the internal power rails.

Expected rails:

- 15 V
- 5 V
- 3.3 V

Measure these rails with a multimeter.

If any rail is missing or significantly incorrect, power should be removed immediately.

---

# Step 5 — Gate Driver Verification

Before connecting a motor, verify that the gate driver behaves correctly.

Apply control signals to the board:

- PWM
- DIR
- ENABLE

Observe the MOSFET gate signals using an oscilloscope.

Verify:

- high-side and low-side gate transitions occur as expected
- deadtime is present between switching events
- gate voltage reaches approximately 15 V

At this stage the motor outputs should remain **unloaded**.

Testing the inverter without a load is sufficient to confirm that the switching behavior is correct.

---

# Step 6 — Check Phase Node Behavior

With the oscilloscope, observe the phase outputs (MOTA, MOTB, MOTC).

The phase nodes should switch cleanly between:

- ground
- the DC bus voltage

Verify that switching occurs without abnormal ringing or instability.

---

# Step 7 — Motor Connection

Once switching behavior is verified, connect a BLDC motor to the three phase outputs.

Connect hall sensors if the external controller uses them for commutation.

Start with a **low bus voltage** such as 12–24 V.

Gradually increase the PWM duty cycle while monitoring:

- supply current
- MOSFET temperature
- system stability

---

# Step 8 — Increase Bus Voltage

After confirming stable operation at low voltage, the bus voltage can be gradually increased toward the intended operating range.

Nominal battery voltage:

48 V

Fully charged battery voltage:

≈56 V

Increase voltage slowly while monitoring current and thermal behavior.

---

# Step 9 — Monitor Thermal Behavior

During early testing, periodically check MOSFET temperatures.

Signs of problems include:

- rapid temperature rise
- uneven heating between MOSFETs
- unexpected current draw

If any abnormal behavior occurs, power down and inspect the switching waveforms.

---

# General Safety Notes

Motor controllers operate with high currents and fast switching edges.

Always:

- start testing at low voltage
- use current-limited supplies
- keep hands away from exposed conductors
- ensure adequate cooling for high-current tests

Following a gradual bring-up process greatly reduces the chance of damaging the controller or connected equipment.

