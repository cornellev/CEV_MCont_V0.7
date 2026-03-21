# Layout Lessons

This document describes the PCB layout principles used when designing the DRV8353FH motor controller. During development it became clear that **PCB layout has a much larger impact on performance than the schematic alone**. Many of the early problems encountered in this project were ultimately caused by parasitic inductance and poorly controlled current return paths.

The following lessons summarize what worked, what failed, and what should be prioritized when designing high-current motor controllers.

---

# Gate Drive Loop Inductance

One of the most important lessons was that **gate drive loop inductance dominates switching behavior**.

The gate drive loop consists of:

- gate driver output
- gate resistor
- MOSFET gate
- MOSFET source
- return path back to the driver

If this loop has significant inductance, several problems occur:

- slower switching
- excessive ringing on the gate node
- reduced effective gate drive strength
- increased switching losses

To minimize this inductance, the gate trace and its return path must be **tightly coupled**.

---

# Use a Solid Ground Plane

All gate drive traces in this design are routed directly over a **continuous ground plane**.

This provides:

- a controlled return path
- low loop inductance
- improved signal integrity

Interrupting the ground plane between the driver and MOSFET can significantly degrade switching performance.

During layout, care was taken to ensure the ground plane remained continuous beneath all gate drive traces.

---

# Keep Gate Driver Close to MOSFETs

Ideally the gate driver should be placed **as close as possible to the MOSFETs**.

Short gate traces reduce inductance and improve switching performance.

In this revision of the controller, all six gate drive outputs originate from the DRV8353 driver chip. Because the driver is centralized, the MOSFETs are approximately **1.5 inches away from the driver**.

This distance increases gate loop inductance and limits switching performance.

Later revisions of the motor controller solve this problem by placing **external gate drivers directly next to the MOSFETs**.

---

# Kelvin Source Routing

High-side gate drivers reference the MOSFET source node, which is also the switching node.

If the driver senses the source voltage through a path that carries large switching currents, the driver reference point will move due to inductive voltage drops.

This can cause:

- incorrect gate voltage
- switching instability
- additional ringing

To prevent this, **Kelvin source routing** was used.

Separate traces connect the driver reference pins to the MOSFET source nodes:

SHA
SHB
SHC

These traces carry almost no current and therefore provide an accurate voltage reference for the gate driver.

---

# Minimize High Current Loop Area

The high current switching loop in a half bridge consists of:

- high-side MOSFET
- low-side MOSFET
- DC link capacitor

When current commutates between MOSFETs, the loop formed by these components must be as small as possible.

Large loop areas increase:

- parasitic inductance
- voltage overshoot
- EMI

In this design the MOSFETs and DC link capacitors were placed to keep this loop compact.

---

# Separate Power and Logic Domains

Motor controllers contain both high-current switching circuitry and sensitive control electronics.

Noise coupling from the power stage into logic circuits can cause erratic behavior.

To reduce this risk:

- logic signals were routed away from switching nodes
- dedicated power rails were used for logic
- a switching regulator generates 15 V for the gate driver
- linear regulators generate clean 5 V and 3.3 V rails

This architecture helps isolate the control electronics from switching noise on the motor bus.

---

# DC Link Capacitor Placement

The DC link capacitors supply current during switching transitions and must therefore be placed very close to the MOSFET bridge.

If these capacitors are located far from the switching devices, parasitic inductance increases and large voltage spikes can occur.

In this design two **470 µF capacitors** were placed near the MOSFET bridge to reduce ripple current and stabilize the DC bus.

Earlier prototypes used smaller capacitance and experienced significant heating due to ripple current.

---

# Gate Resistor Placement

Gate resistors should be placed **as close to the MOSFET gate as possible**.

This ensures the resistor damps ringing in the gate loop rather than allowing oscillations along the trace.

Placing the resistor near the driver instead of the MOSFET reduces its effectiveness.

---

# Avoid Routing Over Switching Nodes

Large copper regions connected to switching nodes can capacitively couple noise into nearby signals.

Sensitive traces should avoid passing over:

- switch nodes
- phase nodes
- MOSFET drains

Keeping logic routing away from these nodes reduces EMI and improves system stability.

---

# Thermal Layout Considerations

MOSFET thermal performance depends heavily on PCB copper area.

Large copper pours connected to the MOSFET drain and source pads help spread heat across the board.

Additional thermal vias can further improve heat conduction into internal layers.

Thermal simulations performed during development helped determine where additional copper area was most beneficial.

---

# Key Takeaways

The most important lessons from the layout process were:

- gate drive loop inductance must be minimized
- the ground plane must remain continuous beneath gate traces
- MOSFETs and DC link capacitors should form a tight switching loop
- Kelvin source routing improves driver stability
- power and logic circuits should be physically separated

Most switching problems encountered during development were ultimately traced back to layout issues rather than schematic errors.

Careful attention to parasitic inductance and current return paths is essential when designing high-current motor controllers.

