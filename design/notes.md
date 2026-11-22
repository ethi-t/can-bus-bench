# CAN read-up notes (M1) — answer in your own words

Sources: TI SLOA101B "Introduction to the Controller Area Network (CAN)"; ESP-IDF TWAI page (Overview + Hardware Connection). Links in `00_Programme/resources.md`.

1. What do CAN-H and CAN-L do in the recessive and dominant states? Why does a differential pair shrug off noise that hits both wires equally?
2. Why is there a 120 Ω resistor at each end of the bus, and what resistance should you measure across CAN-H/CAN-L on a correctly terminated bus (power off)?
3. Sketch a standard (11-bit ID) data frame and label each field. Which field carries the message priority?
4. Two nodes start transmitting at the same instant. How does arbitration decide who wins, and why doesn't the loser's message get corrupted?
5. What is the ACK slot for? Predict what a transmitter sees if no other node on the bus is able to acknowledge.
6. On the ESP32, which part is the CAN controller and which is the transceiver (SN65HVD230)? Which two ESP32 pins connect to the transceiver?

Bonus: what is bit stuffing, and why is it needed?

---

## Learner answers — 2026-11-20 (independent, open-book after the read-up guide)

**Q1.** In a recessive state they have equal voltage, in dominant there is a difference. The wires are twisted together, essentially making their theoretical position the same, so interference interacts equally with them, not changing the difference between them, which is what's being read.

**Q2.** The resistor absorbs waves at either end of the CAN bus; if not there it would have partial reflections which would interfere with readings on the bus.

**Q3.** SOF 1 | Identifier 11 | RTR 1 | IDE 1 | r0 1 | DLC 4 | Data 64 | CRC + delim 15+1 | ACK + delim 1+1 | EOF 7 | IFS 3

**Q4.** The two voltages interact: dom + dom and dom + rec both still result in a difference between CAN-H and CAN-L, whereas rec + rec has no difference; therefore when a receiver receives a different signal than the transceiver is emitting, the node stops transmitting and waits for the bus to be free to retry.

**Q5.** Transmitting nodes leave the ACK slot recessive so any node receiving it can change it to dominant, so the transmitting node can confirm that at least one node received its message successfully. Transmitter sees recessive (1), i.e. 2.5 V on CAN-H and CAN-L.

**Q6.** The ESP32 contains a CAN controller; 3.3 V and GND connect to the transceiver.

**Bonus.** Bit stuffing is reversing a state after receiving it x times in a row; all nodes know this and can therefore interpret the message, but an unused CAN bus won't be recessive forever so you can tell when it's dormant and actually in use.

### Tutor feedback (2026-11-20)
| Q | Mark | Done well | Missing |
|---|---|---|---|
| 1 | 0.8 | Equal/different; common-mode cancellation via twisting | Direction and size: H up ≈3.5 V, L down ≈1.5 V, ≈2 V difference; recessive ≈2.5 V |
| 2 | 0.5 | Absorbs the wave, stops reflections | Why 120 Ω (matches cable Z0); the measured value on a correct bus |
| 3 | 0.8 | All fields, correct order and sizes | Data is 0–64; which field carries priority not stated |
| 4 | 0.6 | Dominant wins (wired-AND); loser detects mismatch, stops, retries | Who wins (lower ID: first to send dominant where they differ); why the winner's frame is intact |
| 5 | 0.6 | ACK purpose exactly right; sees recessive, 2.5/2.5 V | What the transmitter does next (ACK error → retransmit → TEC) |
| 6 | 0.4 | Controller is inside the ESP32 | 3V3/GND are power, not the signal pins: TWAI TX and RX (GPIOs chosen in software) |
| Bonus | 0.4 | Nodes know the rule; link to telling idle from active | Insert 1 opposite bit after 5 identical (transmitter adds, receivers remove); main reason is edges for resynchronisation (no clock wire). Parked with hex |

### Fix-ups after hint 1 (2026-11-20, guided)
- Q1: CAN-H up to 3.5 V, CAN-L down to 1.5 V ✔
- Q2: matching the impedance means all is absorbed; the line doesn't "see" the resistor as an end of the wire ✔. Measured value still not given.
- Q3: identifier; lower number wins ✔
- Q4: the lower ID wins; the winner sees exactly what it sent ✔
- Q5: TEC goes up, eventually disconnected (bus-off) so it stops hogging the bus. Partly ✘: a lone node retransmits forever but stops at error-passive (ISO 11898-1: an ACK error while error-passive doesn't increment TEC). To verify on the M2 bench.
- Q6: not yet answered.
- Q2 (after hint): 60 Ω ✔ (two 120 Ω in parallel)
- Q6 (after hint): TWAI TX GPIO and RX GPIO ✔

**Read-up M1 complete 2026-11-20.** Independent first pass ≈59%; all six correct after hint 1, except Q5's bus-off prediction (corrected; verify on M2 bench). Parked to 2026-11-21: binary/hex mini-lesson, C6 (arbitration in binary), C7 and bonus (bit stuffing).

## Parts arrival check (2026-11-20, from photos in `hardware/photos/`)
| Part | Status | Note |
|---|---|---|
| LM2596S buck ×2 (Weewooday, ASIN B08JQB3NSR) | ✔ | Input/output caps marked 220 µF **35 V**: the module's real input limit is < 35 V, not the chip's 40 V. Take into the M3 TVS sizing (1.5KE27A clamps up to ≈37.5 V at full surge current) |
| OBD-II pigtail (CERRXIAN) | ✔ | Wire colours to be checked with the meter at the bench step |
| Inline mini blade fuse holders ×10 (DIGITEN) | ✔ | |
| 1 A mini blade fuses ×10 | ✔ | Mini size matches the holders |
| 24 MHz 8-ch logic analyser | ✔ | |
| 1N5819 ×5 (Rapid 47-2566) | ✔ | |
| 1.5KE27A TVS (Rapid 16-8001) | ⏳ stock to follow | Qty 20 = Rapid's minimum order (learner, 2026-11-20); not needed until M3 (13 Dec) |
| CAN transceiver boards ×2 (Orivin CP119092, Amazon B0FHGZSV1K) | ✔ | Listing confirms **SN65HVD230**, 2 boards (3.3 V part ✔). Listing's '6–36 V supply' is wrong for this chip: power from the ESP32 **3.3 V** pin only (SN65HVD230 supply 3.0–3.6 V). 120 Ω fitted? check with meter |
| Jesverty SPS-3010V PSU | ✔ (learner report, no photo yet) | Check the input selector is on 230 V before first plug-in |
| 2 × ESP32, breadboard, jumpers, soldering kit, strippers, insulation tape | ✔ owned (learner report) | Both marked plain 'ESP32' (original ESP32 has TWAI ✔) |
| Twisted pair (old Cat5/6 lead) | ⬜ to find | Any round Ethernet patch lead works; use one pair (e.g. orange + white/orange) |
| Multimeter | ⬜ not yet available | Ideally by ~30 Nov (M2 bench checks), at the latest before M3 power-stage work (~7 Dec). Needed for the 120 Ω/60 Ω check, OBD continuity and setting the buck to 5.0 V |

## Bench-bus sketch (2026-11-20, guided — revised after tutor feedback)
- v1 (`2026-11-20_bench-bus-sketch-v1.jpg`): 4 links per node correct; CH/CL board to board; 120 Ω across H–L ✔. Terminators drawn mid-cable, no inter-node GND, no pin numbers.
- v2 (`2026-11-20_bench-bus-sketch-v2.jpg`): 120 Ω at each physical end ✔; pin table GPIO21→CTX, GPIO22→CRX, 3V3, GND ✔; CH solid / CL dashed, "twisted pair ~1 m" ✔; long loop between ESP32s = common GND (label it). Still to add in the KiCad version: GND label on the loop, node A/B names, USB to laptop, logic-analyser probe (CH0 on an RX pin + GND); 120 Ω shown on the boards (R2).
- v3 (`2026-11-20_bench-bus-sketch-v3.jpg`): nodes A/B named, GND link between ESP32s labelled, pin table ✔. Right-hand 120 Ω drawn well past node B (the real R2 sits on B's board, so B is the line end). Still missing: USB to laptop, logic-analyser probe. Sketch accepted as the basis for the KiCad schematic.
- v4 (2026-11-21, `2026-11-21_bench-bus-sketch-v4.jpg`): laptop USB to both ESP32s added; "twisted pair ~1 m between nodes" ✔. Terminators still drawn a short way past each node (in KiCad put them on the boards). Only item left: logic-analyser probe (CH0 on an RX pin + GND). Final hand sketch → KiCad next.
- v5 (2026-11-21, `2026-11-21_bench-bus-sketch-v5.jpg`): logic-analyser CH0 tapped on node A's RX line (branch marked on the GPIO22/CRX row), analyser USB to laptop ✔. Analyser ground lead present but unlabelled — label in KiCad. **Hand sketch final**; next: KiCad bench schematic (plan step 0a).

## M2 step 1 — termination resistance checks (2026-11-22, unpowered, LASER DMM 200 Ω range)
Predictions first (S34, independent): bus with both terminators 60 Ω (120 ∥ 120); one terminator missing 120 Ω; probe position along ~1 m pair makes no difference at DC (wire R ≈ 0).

| Measurement | Raw | − leads 0.4 Ω | Prediction | Photo |
|---|---|---|---|---|
| Lead check (probes shorted) | 0.4 Ω | — | 0.1–0.5 Ω | `hardware/photos/2026-11-22_meter-lead-check-0.4ohm.jpg` |
| Board A CANH–CANL (2000 Ω range) | 120 Ω | ≈120 Ω | 120 Ω | `hardware/photos/2026-11-22_board-A-CANH-CANL-120ohm.jpg` |
| Second board CANH–CANL (200 Ω range; assumed board B) | 120.2 Ω | 119.8 Ω | 120 Ω | `hardware/photos/2026-11-22_single-board-120.2ohm.jpg` |
| Both boards joined, H–H and L–L | 60.2 Ω | 59.8 Ω | 60 Ω | `hardware/photos/2026-11-22_bus-both-terminators-60.2ohm.jpg` |

Result: both boards have their 120 Ω terminator ("121" SMD) fitted; bus reads half of one terminator, as predicted. Check: 119.8 ∥ 120 ≈ 59.9 Ω vs 59.8 measured (within meter resolution).

## KiCad bench schematic (M2 step 0a, 2026-11-22)
- Project `hardware/Kicad/can_bench_bus` (KiCad 10). Exports: `design/can_bench_bus.pdf`, `design/can_bench_bus.svg`.
- Nets: A_CTX U1 IO21→U3 D; A_CRX U1 IO22→U3 R→J1 CH0; B_CTX/B_CRX same on U2/U4; CAN-H/CAN-L with R1, R2 120 Ω at the ends; +3.3V to all VDD/VCC; GND to all grounds, Rs ×2, J1 pin 2.
- Decisions: Rs → GND (high-speed mode; Rs high = standby, driver off → node can't transmit or ACK). Vref = VCC/2 output, unused → no-connect. Unused ESP32 pins + EN (pulled up on dev board) → no-connect. PWR_FLAG on +3.3V and GND (supply from USB via dev-board regulator, off-sheet).
- Review path: first saved netlist already correct; ERC 81 → 0. Mistake kept: Rs and Vref first tied to 3V3.
- To do: title block (title, date, rev A) and re-plot.
