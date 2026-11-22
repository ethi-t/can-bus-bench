# CAN bus project — stage plan (written 2026-11-18)

Budget: ~9 h/week project time. Parts list and links: `hardware/BOM.md`. Requirements R1–R5 and safety rules: `README.md`.

| Stage | Dates | ≈ h | Parts needed |
|---|---|---|---|
| M1 Order + study | 18–22 Nov | 5 | none (place orders) |
| M2 Bench bus + documentation start | 23 Nov – 6 Dec | 20 | 2× CAN board, 2× ESP32, twisted pair, logic analyser, breadboard/jumpers, multimeter; KiCad, Falstad/LTspice, Git |
| M3 Power stage | 7–13 Dec | 8 | LM2596, fuse holder + 1 A fuse, 1N5819, 1.5KE27A, OBD cable, **current-limited bench PSU (not owned)** |
| M4 Car, listen-only | 14–31 Dec | 20 | all of the above; car node with its 120 Ω removed; laptop |
| M5 Decode + GitHub polish | 1–17 Jan | 12 | none (Python, KiCad) |
| M6 Report + EMC | 18–24 Jan | 8 | none |

Application submission target 12 Dec falls in M3 week: keep M3 light that week; use M2 results in the personal statement.

**Change 2026-11-20 (learner decision):** documentation moved forward from M5 into M2, so that by 12 Dec there is something real to show — GitHub repo with history, KiCad bench schematic, build log, logic-analyser trace, soak log, termination simulation. M2 grows 16 → 20 h; M5 shrinks 16 → 12 h (bench schematic and repo already exist by then).

## M1 — Order + study (18–22 Nov)
- Place the three orders (Pi Hut, Amazon, Rapid).
- Install PlatformIO (or Arduino IDE with ESP32 core); confirm both ESP32 models support TWAI (not C2).
- Create a Git repo for `firmware/`; first commit = README + this plan.
- Study: differential signalling, why 120 Ω at each end (cable characteristic impedance, reflections), dominant/recessive bits, frame fields (SOF, 11-bit ID, RTR, DLC, data, CRC, ACK, EOF), arbitration, bit stuffing, error counters (TEC/REC, error-passive, bus-off). Write one page of notes in `design/`.
- Evidence: notes + a hand-drawn frame diagram.

## M2 — Bench bus + documentation start (23 Nov – 6 Dec)
0. Documentation setup (first session, ~1 h): create the GitHub repo (carried over from M1) with README + this plan; start `docs/build-log.md`. Every bench session gets a dated entry: **prediction → what I did → what I measured → photo/screenshot → what went wrong / what I'd change**. Mistakes stay in.
0a. KiCad schematic of the bench bus (~3 h), replacing the notebook sketch (2026-11-20): two ESP32 + SN65HVD230 nodes, 3V3/GND/CTX/CRX with GPIO numbers, CAN-H/CAN-L twisted pair, 120 Ω at each physical end (on the boards), common GND between nodes, logic-analyser probe point. Export PDF/PNG to `design/`. Must match the firmware pin numbers.
0b. Termination simulation (~2 h, Falstad or LTspice transmission line; EMC0 tie-in): pulse into a line with 120 Ω both ends / one end open / wrong value (e.g. 60 Ω, 1 kΩ). Predict first; screenshot each; compare with break-it experiment 6 on the bench.
1. Arrival checks: photograph parts; measure each board's CAN-H↔CAN-L resistance unpowered (~120 Ω); after wiring both, the bus should read ~60 Ω. Predict first.
2. Wire each board: 3V3, GND, TX→GPIO, RX→GPIO; join CAN-H/CAN-L with the twisted pair. Common ground between nodes.
3. Firmware v1: TWAI normal mode, 500 kbit/s. Node A sends a counter frame every 100 ms; node B prints received frames and echoes with a different ID.
4. R1 test: 10-minute soak, log frame counts + TEC/REC from both nodes → `test/`.
5. Logic analyser on the RX pin; PulseView CAN decoder; identify ID, DLC, data, CRC, ACK slot. Screenshot for the report.
6. Break-it experiments (predict each first): remove one terminator; mismatched bit rates; unplug node B (transmitter gets no ACK → error counters climb).
7. Listen-only on the bench: put node B in listen-only. Observe that A now gets no ACK — explains why the car is fine (other ECUs ACK) and is the R2 code path. Add boot check that prints the mode.
- Milestone evidence (all in the repo by 6 Dec): KiCad schematic, build log, soak log, annotated analyser trace, termination simulation vs bench comparison, short write-up. Feed into the personal statement (12 Dec) and interview prep.

## M3 — Power stage (7–13 Dec) — needs bench PSU
- Before buying: choose a current-limited 0–30 V / 0–3 A PSU (Stage 1 purchase in `00_Programme/equipment-software.md`).
- Design review (short): fuse rating vs. expected current; 1N5819 drop; TVS sizing against the LM2596's 40 V limit (standoff above 14.4 V charging voltage, clamp below 40 V) — decide whether 1.5KE27A is fitted.
- OBD cable: continuity-check pins 4, 5, 6, 14, 16; label leads; insulate unused wires individually.
- Bench test with PSU current limit ~200 mA: set buck to 5.0 V at 12 V in **before** connecting an ESP32. Sweep input 6–16 V; record output voltage, input current; compute efficiency. Find the input voltage where the ESP32 browns out (cranking simulation).
- Evidence: table + plot (Python), schematic sketch.

## M4 — Car, listen-only (14–31 Dec)
- Desolder the 120 Ω on the car node's transceiver board; re-measure.
- Rules: parked, engine off first, then ignition on, then engine running; cables clear of pedals; never transmit; unplug after each session.
- Firmware: listen-only, try 500 kbit/s then 250 kbit/s; print timestamped frames over serial; Python logger writes CSV.
- Findings: bit rate, number of IDs, frame rates per ID; whether a gateway blocks traffic (little traffic = finding, not failure).
- Optional later (parked only, separate firmware, design review first): standard OBD-II PID requests (RPM 0x0C, speed 0x0D, coolant 0x05).

## M5 — Decode + GitHub (1–17 Jan)
- CP1 L04 (files/CSV) feeds this. Load logs in Python; frequency of each ID; find signals that change with RPM/speed (or decode PID replies).
- Plot ≥3 signals vs time (R4).
- Extend the M2 KiCad schematic with the power stage and car node (120 Ω removed); polish the repo README with photos and plots.

## M6 — Report + EMC (18–24 Jan)
- One-page report: requirements → verification table (R1–R5 pass/fail with evidence).
- "EMC in this design": differential signalling and common-mode rejection, termination and reflections, twisted pair, automotive transients (load dump, cranking), TVS/fuse. Ties to EMC0.
- 5 interview talking points (problem, decision, measurement, mistake, what next).
