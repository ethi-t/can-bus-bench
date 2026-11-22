# Project 1 — CAN Bus Interface (ESP32)

**Window:** 2026-11-17 → end Jan 2027 (~9 h/week) · **Status:** 🟨 M1 done 2026-11-20 (parts arrived, read-up done); next M2 bench bus + documentation start (GitHub repo, KiCad schematic, build log, termination sim; due 6 Dec); multimeter owned 2026-11-22
**Interview value:** real hardware + firmware + measurement, documented on GitHub; direct link to
EMC (differential signalling) and to automotive/defence vehicle buses.

## Goal
Build an ESP32 CAN interface that (1) works on a two-node bench bus, then (2) passively logs and
decodes traffic from the car's OBD-II port, with data plotted in Python.

## Requirements (draft — refine in a design review)
| ID | Requirement | Verification |
|---|---|---|
| R1 | Two ESP32 nodes exchange CAN frames at 500 kbit/s on the bench, error-free over 10 min | Frame counters, error counters logged |
| R2 | Car connection is **receive-only** (TWAI listen-only mode): the node never transmits on the car's bus | Code review + mode check at boot |
| R3 | Powered from the OBD-II 12 V pin via fuse + wide-input buck, survives cranking dips/transients without resetting | Log brown-outs during engine start |
| R4 | Logs timestamped frames to serial → Python → CSV; plot at least 3 decoded signals (e.g. RPM, speed, coolant) | Plots in report |
| R5 | Total cost ≤ £30 | BOM |

## Parts (≈ £18–34 core) — full order list: `hardware/BOM.md`
- 2 × SN65HVD230 3.3 V CAN transceiver boards (check whether the 120 Ω terminator is fitted)
- OBD-II breakout cable/connector (pins: 6 CAN-H, 14 CAN-L, 16 +12 V battery, 4/5 ground — confirm for your car)
- 12 V → 5 V buck module with input rating comfortably above 12 V (≥ 30 V preferred, for transients)
- Inline blade fuse holder + 1 A fuse; optional TVS diode on the 12 V input
- Twisted pair wire for the bench bus

## Safety and legal
- **Never transmit on the car's bus.** Listen-only mode. Do not connect while driving until it is
  proven stable on the bench and parked; tidy cables so nothing interferes with pedals or controls.
- Optional later phase: standard OBD-II PID requests (what any scan tool does) **only while parked**.
  Never send arbitrary/unknown frames.
- Fuse the 12 V feed at the connector. The OBD-II pin 16 is permanently live: unplug when finished
  (battery drain).
- Some cars have a gateway on the OBD port; you may see little traffic without requests. That's a finding, not a failure.

## Milestones
| # | Milestone | Target | Status |
|---|---|---|---|
| M1 | Parts ordered; CAN physical layer + frame format study (EMC0 L5 tie-in) | 2026-11-22 | ✅ ordered 2026-11-18, read-up + parts arrived 2026-11-20 (TVS to follow) |
| M2 | Bench bus: two ESP32s, frames both ways, logic-analyser trace; GitHub repo + build log started, KiCad bench schematic, termination simulation | 2026-12-06 | 🟨 KiCad schematic done 2026-11-22 (ERC 0/0, PDF + SVG in design/); bench 60 Ω check done 2026-11-22; GitHub repo + build log entry 1 2026-11-22 |
| M3 | Power stage built and tested on the bench PSU at 12–14.4 V (current-limited) | 2026-12-13 | ⬜ |
| M4 | Car: listen-only logging, bit rate found, frames captured | 2026-12-31 | ⬜ |
| M5 | Python decoding + plots; full schematic (power stage, car node); repo README polish | 2027-01-17 | ⬜ |
| M6 | One-page report incl. "EMC in this design" (EMC0 evidence); interview talking points | 2027-01-24 | ⬜ |

## Plan
Stage-by-stage plan with parts per stage: `plan.md`.

## Folders
`design/` calculations, schematic, simulations · `docs/` build log · `hardware/` photos, wiring · `firmware/` ESP32 code (Git) · `test/` logs, plots
