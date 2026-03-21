# Power Stage Design

This document describes the power stage design methodology used for the DRV8353FH motor controller. The goal of this analysis was to estimate MOSFET heating and determine which design parameters most strongly affect efficiency and thermal performance.

---

# Loss Model

The MOSFET power dissipation model used during development was:

P_loss = (I_rms^2)*R_DS_ON
       + (V_in * I_out / 2) * (t_on + t_off) * f_sw
       + V_in^2 * C_oss * f_sw
       + V_G^2 * C_GS * f_sw

Each term represents a different physical mechanism.

---

# Conduction Loss

The first term represents MOSFET conduction loss:

P_cond = I_rms^2 * R_DS(on)

For the MOSFET used in this design:

R_DS(on) ≈ 1.8 mΩ

Example calculation at 50 A RMS:

P_cond = 50^2 * 0.0018

P_cond ≈ 4.5 W per MOSFET

Because only two MOSFETs conduct during each commutation state, conduction loss is the dominant loss component at low switching frequencies.

---

# Switching Loss

Switching loss occurs during the interval where both voltage and current are present across the MOSFET.

P_switch = (V_in * I_out / 2) * (t_on + t_off) * f_sw

Measured switching characteristics:

- Miller plateau duration: 100–150 ns
- total switching time: 200–300 ns

Example estimate:

V_in = 56 V

I_out = 50 A

(t_on + t_off) ≈ 250 ns

f_sw = 5 kHz

P_switch ≈ 1.75 W per MOSFET

This shows that switching loss is smaller than conduction loss at the chosen switching frequency.

---

# Output Capacitance Loss

The MOSFET output capacitance must be charged and discharged each switching cycle.

P_Coss = V_in^2 * C_oss * f_sw

Although typically smaller than conduction and switching losses, this component becomes more significant at higher switching frequencies.

---

# Gate Drive Loss

Gate drive loss comes from charging and discharging the MOSFET gate capacitance.

P_gate = V_G^2 * C_GS * f_sw

In practice this term is often calculated using total gate charge:

P_gate = Q_g * V_G * f_sw

For the MOSFET used in this design:

Q_g ≈ 51 nC

V_G = 15 V

At 5 kHz:

P_gate ≈ 3.8 mW

Gate drive loss is therefore negligible compared to conduction and switching losses.

---

# Switching Frequency Selection

One of the most important design parameters is switching frequency.

Switching losses scale linearly with switching frequency.

Reducing the switching frequency significantly lowers total MOSFET dissipation.

This design uses:

f_sw = 5 kHz

The lower frequency reduces thermal stress on the MOSFETs while remaining acceptable for the application.

---

# Thermal Modeling

Once the electrical loss model was created, the resulting power dissipation values were used to estimate junction temperatures.

Two approaches were used:

1. Thermal resistance circuit models

2. ANSYS thermal simulations

Both steady‑state and transient thermal analyses were performed.

These simulations helped identify which parameters had the largest effect on MOSFET heating.

---

# Gate Drive Design

The DRV8353FH gate driver allows configurable drive strength.

Final configuration:

Source current = 300 mA

Sink current = 600 mA

Gate resistors:

R_g = 4 Ω

This configuration provided a balance between:

- fast switching
- manageable ringing
- safe driver operation

Measured switching characteristics:

- Miller plateau duration: 100–150 ns
- total switching time: 200–300 ns

Deadtime:

100 ns

---

# Bus Voltage

The inverter is designed for a maximum bus voltage of:

60 V

The vehicle battery is nominally:

48 V

Fully charged voltage is approximately:

56 V

This voltage margin influenced the MOSFET selection.

---

# DC Link Capacitor Design

The DC bus contains:

2 × 470 µF capacitors

Earlier versions of the board used significantly smaller capacitance.

These capacitors experienced significant heating due to ripple current.

Increasing the DC link capacitance reduced ripple current and eliminated the overheating problem.

---

# Observed Performance

Measured peak current on this revision of the board:

50 A bus current

Later revisions with optimized power stages and external gate drivers have reached:

150 A peak bus current

---

# Design Takeaways

Key lessons from the power stage development:

- Conduction losses dominate at low switching frequencies
- Switching loss scales linearly with switching frequency
- Gate drive layout strongly affects switching speed
- Thermal modeling should be done early in the design process

Reducing parasitic inductance and optimizing the gate drive loop were the most important improvements made during development.

