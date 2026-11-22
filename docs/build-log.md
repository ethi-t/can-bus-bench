# Build log — CAN bench bus

Dated entries, newest at the bottom. Each one covers what I did, why, what I measured and what's next.

---

## Entry 1 — 2026-11-20 → 21: bench-bus sketch, v1 → v5

**Goal:** draw the two-node bench bus (2 × ESP32 + 2 × SN65HVD230 transceiver boards on a twisted pair) by hand before drawing it in KiCad.

| Version | Date | What changed | Sketch |
|---|---|---|---|
| v1 | 2026-11-20 | First layout: 4 links per node, CAN-H/CAN-L board to board, 120 Ω across H–L | [v1](../design/2026-11-20_bench-bus-sketch-v1.jpg) |
| v2 | 2026-11-20 | Terminators moved to the ends of the bus; pin table (GPIO21 → CTX, GPIO22 → CRX, 3V3, GND); twisted pair ~1 m | [v2](../design/2026-11-20_bench-bus-sketch-v2.jpg) |
| v3 | 2026-11-20 | Nodes named A and B; common GND link between the ESP32s labelled | [v3](../design/2026-11-20_bench-bus-sketch-v3.jpg) |
| v4 | 2026-11-21 | Laptop USB to both ESP32s added | [v4](../design/2026-11-21_bench-bus-sketch-v4.jpg) |
| v5 | 2026-11-21 | Logic-analyser CH0 tapped on node A's RX line, analyser USB to laptop. Final hand sketch | [v5](../design/2026-11-21_bench-bus-sketch-v5.jpg) |

### Why the terminators moved

I wanted the nodes on the inside of the resistors: the resistors at the two ends of the cable with
a node attached at each end, not the nodes at the ends of the cable with the resistors somewhere along it.

The resistor is the same as the characteristic impedance of the wire, and therefore absorbs all
waves travelling through the wire, leaving no partial reflections to disrupt other signals. Partial reflections are caused by a sudden
difference in impedance. Therefore, if the resistor was in the centre of the wires and not at the end,
there would still be a difference in impedance at the end of the wire, causing partial reflections and
disrupting the signal.

### Why the nodes need a common ground

The two ESP32s must have a common ground in order to work off the same reference point, so CAN-H and
CAN-L stay within the voltage range each transceiver can read.

### What I'd do differently / what's next

Next: draw the v5 sketch as a KiCad schematic (done 2026-11-22), then wire everything up on a breadboard.

---

## Entry 2 — 2026-11-22: KiCad schematic + termination resistance checks

**Goal:** draw the v5 sketch in KiCad, then check the termination resistance with a multimeter before wiring anything to power.

### Schematic

KiCad 10 schematic of the bench bus (2 × ESP32-WROOM-32, 2 × SN65HVD230, 120 Ω at each end of the bus).
ERC 0 errors / 0 warnings. Title block filled in (rev A, 2026-11-22). Plots: [PDF](../design/can_bench_bus.pdf) · [SVG](../design/can_bench_bus.svg).

### Resistance checks (unpowered)

| Measurement | Meter reading | True resistance (reading − 0.4 Ω leads) | Photo |
|---|---|---|---|
| Probes shorted together (lead resistance) | 0.4 Ω | — | [photo](../hardware/photos/2026-11-22_meter-lead-check-0.4ohm.jpg) |
| CAN-H to CAN-L, one board (one terminator) | 120.2 Ω | 119.8 Ω | [photo](../hardware/photos/2026-11-22_single-board-120.2ohm.jpg) |
| CAN-H to CAN-L, bus with both terminators | 60.2 Ω | 59.8 Ω | [photo](../hardware/photos/2026-11-22_bus-both-terminators-60.2ohm.jpg) |

I measured the resistance of the multimeter leads by touching the probes together, then measured the
resistance across the CAN bus with one resistor connected and with two resistors connected. Subtracting
the lead resistance from each reading gives the true resistance.

### Why about 60 Ω

The resistance is about 60 Ω, as I have two 120 Ω resistors connected in parallel.

### What's next

Termination simulation (Falstad/LTspice), then wire the bus on the breadboard and run firmware v1.
