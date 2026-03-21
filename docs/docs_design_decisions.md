# Design Decisions

This document explains the engineering reasoning behind the architecture of the DRV8353FH motor controller. The goal of this project was not only to produce a working motor controller, but also to document the lessons learned while developing a robust inverter for Shell EcoMarathon vehicles.

---

# Background: Shell EcoMarathon

Since 2022 our team has competed in the **Shell EcoMarathon Urban Concept** category. This competition strongly rewards efficiency, reliability, and simplicity.

Motor controllers are one of the most failure‑prone systems on student vehicles, so one of the goals of this design was to produce a controller that is:

- reliable
- understandable
- reproducible by other teams

This repository is partially intended as a resource for other Shell EcoMarathon teams attempting to build their own motor controllers.

---

# First Generation Motor Controller

Before this design, our team attempted to build a custom motor controller using a software‑controlled deadtime scheme.

This approach had several problems:

- Deadtime was implemented using polling
- Switching timing was inconsistent
- Shoot‑through events occurred
- The system was difficult to debug

When the project was handed over to a new generation of engineers, the decision was made to **delete the entire architecture and start from scratch**.

The new design philosophy was simple:

> Use integrated solutions whenever possible and eliminate unnecessary firmware complexity.

---

# Attempted Sensorless Controller

The first integrated solution evaluated was the **TI MCT8316A**, a sensorless trapezoidal motor controller.

Advantages:

- Fully integrated motor control
- Minimal external components
- Sensorless operation

However, the sensorless approach created a major problem for our application.

Sensorless controllers must **align the rotor before starting commutation**. This produces an initial jerk and unpredictable startup behavior.

For automotive applications this was unacceptable.

Because of this limitation, the design moved to a **sensored architecture using hall sensors**.

---

# Transition to DRV8353FH

The next architecture used the **Texas Instruments DRV8353FH** gate driver.

Reasons for choosing this chip:

- Integrated 3‑phase gate driver
- Hardware configuration mode (no SPI required)
- Built‑in current sense amplifiers
- Fault detection
- Bootstrap high‑side drive

Using the DRV8353 allowed the system to remain mostly hardware‑driven while still providing a robust gate driver.

External control electronics perform **6‑step trapezoidal commutation using hall sensors**, while the DRV8353 handles the gate drive.

---

# Early Power Stage Problems

The initial versions of the power stage suffered from several major issues:

- MOSFET overheating
- shoot‑through
- slow switching

These problems were eventually traced to **gate drive loop inductance and poor return path control**.

At this stage we began studying MOSFET switching behavior in more detail, including:

- Miller plateau behavior
- reverse recovery
- MOSFET capacitances (especially Crss)
- parasitic inductance

This stage of development dramatically improved understanding of power electronics design.

---

# Gate Drive Optimization

One of the largest improvements came from reducing the **gate drive loop inductance**.

Key layout changes included:

- routing gate traces directly over a solid ground plane
- keeping the return path tightly coupled with the gate trace
- avoiding interruptions in the ground plane

For the high‑side drivers, **Kelvin source routing** was used so the driver reference point matched the MOSFET source node.

This corresponds to the **SHA / SHB / SHC nodes** in the schematic.

After these improvements, switching performance improved significantly.

---

# MOSFET Selection Evolution

Early prototypes used:

IRFS7530TRL7PP (Infineon StrongIRFET)

These devices were rated for **60 V VDS**, which was too close to the maximum bus voltage for comfortable operation.

The design later moved to:

IPB017N10N5ATMA1

Key advantages:

- 100 V VDS rating
- low RDS(on)
- strong current capability

Although higher VDS MOSFETs typically have higher RDS(on), the added voltage margin was considered necessary.

Later revisions of the motor controller further optimized MOSFET selection by reducing VDS rating and switching to **TOLL packages**, but those improvements are not present in this revision.

---

# Gate Drive Tuning

Gate drive strength and gate resistors were tuned to balance switching speed and ringing.

Final configuration:

DRV8353 drive strength:

- 300 mA source
- 600 mA sink

Gate resistors:

4 Ω

This produced switching characteristics of approximately:

- Miller plateau duration: 100–150 ns
- Total switching time: 200–300 ns

Deadtime was fixed at **100 ns**, which is the default configuration for the DRV8353 hardware interface variant.

---

# Switching Frequency

The inverter operates at a relatively low switching frequency:

5 kHz

The primary motivation was **reducing switching losses**. Because switching loss scales linearly with frequency, lowering the switching frequency significantly reduces MOSFET heating.

Acoustic noise was acceptable for the vehicle application.

---

# Power Supply Architecture

The gate driver and logic electronics are powered from a dedicated supply derived from the motor bus.

Power stages:

48 V → 15 V buck converter

15 V → LDO → 5 V

15 V → LDO → 3.3 V

Using a switching regulator for the 48 V to 15 V conversion reduces heat dissipation and isolates the gate driver supply from noise on the main bus.

---

# DC Link Capacitors

The board uses:

2 × 470 µF electrolytic capacitors

Earlier versions used much smaller capacitance and experienced significant capacitor heating.

Increasing the DC link capacitance reduced ripple current and significantly improved thermal performance.

---

# Limitations of the Current Architecture

One limitation of the DRV8353 architecture is that **all six gate drive outputs originate from a single chip**.

This forces the driver to be located approximately **1.5 inches away from the MOSFETs**, increasing gate loop inductance.

This inductance limits the maximum usable gate drive strength and switching speed.

Later revisions of the motor controller solve this problem by using **external gate drivers placed directly next to the MOSFETs**.

---

# Later Revisions

Two later revisions of the motor controller introduce additional optimizations:

- external gate drivers
- lower VDS MOSFETs
- TOLL MOSFET packages
- improved power stage layout

These revisions significantly increase the peak current capability of the controller.

However, the board documented in this repository represents the **first stable architecture that successfully drove the motor reliably**.

---

# Key Lessons

The most important lessons from this project were:

- Gate drive layout is critical
- Parasitic inductance dominates switching behavior
- MOSFET selection must consider voltage margin and switching performance
- Thermal modeling should be performed early in the design process

Most importantly:

> Power electronics design is heavily influenced by parasitics and layout. Schematics alone do not determine system performance.

