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

---

## Entry 3 — 2026-11-22: termination simulation (Falstad)

**Goal:** before touching the bench, predict what a fast edge does on a 120 Ω line with different terminations, then check the predictions in a simulator.

### Set-up

Falstad circuit simulator, starting from its "Termination" example (Circuits ▸ Transmission Lines). Four identical lossless lines, Z0 = 120 Ω, one-way delay Td = 10 ns, each driven through Rs = 120 Ω by a 0 → 2 V pulse (10 MHz). Only the far-end load changes: 120 Ω, open (1 GΩ), 60 Ω, 1 kΩ. A scope on each load and a voltmeter probe at the near end (node A) of the open line.

Model limits: ideal lossless line, perfectly sharp edges. A real twisted pair has losses, and the SN65HVD230 has slower edges, so real reflections are smaller and rounder.

### Predictions (worked by hand first)

Formulas used: V⁺ = Vs·Z0/(Rs + Z0) = 1 V; Γ_L = (RL − Z0)/(RL + Z0); v_end = V⁺(1 + Γ_L).

| Load RL | Γ_L | Predicted far-end voltage | Simulated | Match |
|---|---|---|---|---|
| 120 Ω | 0 | 1.0 V | 1 V | ✔ |
| open (1 GΩ) | +1 | 2.0 V | 2 V | ✔ |
| 60 Ω | −1/3 | 0.667 V | 666.667 mV | ✔ |
| 1 kΩ | 11/14 | 1.786 V | 1.786 V | ✔ |

Check: with Rs = Z0, v_end = Vs·RL/(RL + Z0), the plain DC divider, so each answer agrees with the DC value.

Screenshot: [termination sim, 3 ns pulses](../design/2026-11-22_termination-sim-falstad-3ns-pulse.png)

### What I saw at the near end (open line)

The voltage spiked twice at 1 V with a 2Td = 20 ns time gap. The 3 ns long pulse had finished, so node A was back to 0 V long before the echo returns. The two never overlap, so the max voltage can never be 2 V. The wave is absorbed at Rs, and therefore the voltage does not spike a third time.

### What went wrong / what I learned

Source voltage was 0 to 5 V instead of 0 to 2 V, giving me 2.5× my predictions. Changed the source voltage to 0 to 2 V.

### Prediction for the bench (break-it experiment 6)

One bit at 500 kbit/s lasts 2000 ns (1/(500 × 10³) s = 2 µs). A round trip on my 1 m twisted pair is about 10 ns, so a bit is about 200 times longer than a round trip. The echo comes back and is absorbed within the first 1% of the bit. The receiver doesn't sample until well into the bit, so the round trip has already been completed and the receiver reads nothing from the echo. Because the receiver doesn't receive it, the echo doesn't change any bit to dominant, so I predict frames and ACKs will still get through with one terminator removed. I won't be able to see the reflections themselves without an oscilloscope on CAN-H/CAN-L.
