# CAN bus project — order list (M1, order by 2026-11-22)

**Status: ORDERED 2026-11-18; ARRIVED 2026-11-20** except the 1.5KE27A (Rapid, stock to follow, min. order 20). Transceivers came from Amazon (Orivin), replacing the Pi Hut order: chip marking to confirm. Multimeter not ordered: borrow (not yet available). Arrival check: `design/notes.md`.

Prices are rough UK guide prices (eBay/Amazon/AliExpress/The Pi Hut), not quotes. Budget R5: ≤ £30.

## Must order
| # | Qty | Part | What to search for | Why / check on arrival | ≈ £ |
|---|---|---|---|---|---|
| 1 | 2 | 3.3 V CAN transceiver board | "SN65HVD230 CAN module" (also sold as VP230; Waveshare version is fine) | One per node. Most have a 120 Ω terminator (R2) fitted: right for the bench bus (both ends terminated), but **remove/cut it on the car node** (car bus is already terminated) | 3–6 (pair) |
| 2 | — | ~~Second ESP32 dev board~~ | **Already owned (2026-11-18)** — check it isn't an ESP32-C2 (no TWAI) | R1 needs two nodes | 0 |
| 3 | 1 | OBD-II male plug, open-ended cable | "OBD2 16 pin male pigtail cable" / "OBD2 male connector open end" | Avoid OBD-to-DB9 cables (pinouts vary). Verify pins 6 (CAN-H), 14 (CAN-L), 16 (+12 V), 4/5 (GND) with the multimeter before use | 4–8 |
| 4 | 1 | 12 V → 5 V buck module, ≥ 30 V input | "LM2596 DC-DC buck module" (40 V max input; adjustable — set to 5.0 V on the bench **before** connecting the ESP32). Fixed-5 V versions also fine | Feeds the ESP32 5 V/VIN pin from OBD pin 16; survives charging-system highs. Mini-360 / MP1584 boards are too close to their input limit — skip | 2–4 |
| 5 | 1 | Inline blade fuse holder + 1 A fuses | "inline mini blade fuse holder" + "1A mini blade fuse" (match holder size: mini or standard ATO) | Fuse at the connector end of the 12 V lead | 3–5 |
| 6 | ~2 m | Twisted pair | Offcut of Cat5/Cat6 network cable (one pair = CAN-H/CAN-L) | Bench bus; free if you have an old patch lead | 0–2 |

**Core total ≈ £18–34** (the second ESP32 is what pushes it towards the £30 cap).

## Optional (recommended)
| Part | What to search for | Why | ≈ £ |
|---|---|---|---|
| TVS diode on 12 V input | 1.5KE27A (unidirectional, through-hole) | Clamps load-dump/transient spikes below the LM2596's 40 V limit. Final choice is a sizing exercise in the M3 design review (EMC0 tie-in) — don't fit until then | 1–2 |
| Reverse-polarity diode | 1N5819 Schottky (in series with the 12 V feed) | Protects against a swapped lead on the bench PSU | <1 |
| 8-ch USB logic analyser, 24 MHz | "24MHz 8 channel logic analyzer" (works with sigrok/PulseView) | M2 trace of the TX/RX lines at 500 kbit/s; reused later for LoRa SPI | 8–12 |

## Already owned / needed from the kit
ESP32 #1 + USB cable (a second data-capable USB cable for ESP32 #2), breadboard, jumper wires, multimeter.
M3 also needs a **current-limited bench PSU** (Stage 1 purchase in `00_Programme/equipment-software.md`) — not in this budget.

## Chosen sources (checked 2026-11-18; prices change)
Second ESP32 and USB cable already owned, so the freed budget moves the logic analyser into the order.

| Part | Seller | Link | Price seen |
|---|---|---|---|
| 2 × Waveshare CAN Board (SN65HVD230) — **qty 2** | The Pi Hut | https://thepihut.com/products/can-board-sn65hvd230 | £4.50 each inc VAT, in stock. 120 Ω terminator is fixed (reviews): desolder it on the car node |
| LM2596S buck, 4–40 V in, voltmeter, reverse-polarity protected (2 pcs) | Amazon UK | https://www.amazon.co.uk/dp/B08JQB3NSR | £9.99 (4.5★, 172) |
| CERRXIAN OBD-II 16-pin male → open pigtail, 1.5 m (pin 6 green, 14 brown/white, 16 green/white, 4 orange, 5 yellow — verify with meter) | Amazon UK | https://www.amazon.co.uk/dp/B0CZQLWGXC | £10.99 (4.7★, 14) |
| DIGITEN inline mini (ATM) blade fuse holder ×10 | Amazon UK | https://www.amazon.co.uk/dp/B00VLBAHX2 | £5.99 (4.6★, 125) |
| 1 A mini blade fuses ×10 | Amazon UK | https://www.amazon.co.uk/dp/B09GYBBXL7 | £3.99 (4.8★, 30) |
| 24 MHz 8-ch USB logic analyser (sigrok/PulseView) | Amazon UK | https://www.amazon.co.uk/dp/B08JV4W1CD | £10.99 (4.4★, 47) |
| Littelfuse 1.5KE27A TVS (don't fit until M3 review) | Rapid Electronics | https://www.rapidonline.com/littelfuse-1-5ke27a-tvs-diode-do201-28-4v-1500w-surge-protection-diode-16-8001 | ≈ £0.57 ex VAT |
| ST 1N5819 Schottky (multiples of 5) | Rapid Electronics | https://www.rapidonline.com/st-1n5819-schottky-diode-40v-1a-do41-47-2566 | £0.08 each ex VAT |
| Twisted pair | — | Cut a pair from any old Cat5/6 patch lead | 0 |

Total ≈ £52 + Rapid delivery (check at checkout). Over the original £30 R5 cap because of the logic analyser
and the multi-packs (spare bucks/fuse holders reused on LoRa and drone projects); without the analyser ≈ £41.

## Bench PSU (needed before M3, 7 Dec) — checked 2026-11-18
| Option | Seller | Link | Price seen | Notes |
|---|---|---|---|---|
| **Recommended:** Jesverty SPS-3010V, 0–30 V 0–10 A switch-mode | Amazon UK (sold by JIUYUAN GLOBAL) | https://www.amazon.co.uk/dp/B0DDPVDWPZ | £56.99 (4.6★, 985) | Encoder set V and I limit, output on/off button, OCP. **Has a 115/230 V input selector — check it's on 230 V before first plug-in** |
| Buy-once alternative: Voltcraft LPS1305, 0–30 V 0–5 A linear | Rapid Electronics | https://www.rapidonline.com/voltcraft-lps1305-bench-psu-single-output-0-30vdc-0-5a-150w-64-9441 | £126.63 ex VAT (≈ £152 inc) | Low-noise linear; better later for analogue/EMC measurements |
| Not chosen | — | Korad KD3005D (Amazon, third-party seller, £162.66); GW Instek GPS-3030DD (Rapid, £206 ex VAT); Aim-TTi EL302R (Rapid, out of stock until 8 Dec) | | |

## Multimeter (not owned — learner will **borrow** one, 2026-11-18; not ordering). Needed M2 (from ~23 Nov) to M4; it needs a fused mA/A range, or use the PSU's current display in M3. Fallback if borrowing falls through:
| Part | Seller | Link | Price seen |
|---|---|---|---|
| UNI-T UT61E+ true-RMS, CAT III, fused current ranges, USB logging | Amazon UK (sold by Amazon EU) | https://www.amazon.co.uk/dp/B08NDLYLY3 | £87.19 (4.4★, 58; only 2 left) |

## Complete-project check (2026-11-18)
Tab group = CAN boards, buck, OBD cable, fuse holders, 1 A fuses, logic analyser, 1.5KE27A, 1N5819, Jesverty PSU, UT61E+.
Already owned (learner confirmed): 2 × ESP32 + USB cables, soldering kit, breadboard + jumpers, wire strippers / heat-shrink / tape / cable ties / old Cat5. Software (PlatformIO, PulseView, Python, KiCad) is free. Plus a laptop and the car.

## On arrival
1. Photograph each part → `hardware/`.
2. Check the transceiver's R2 (120 Ω) with the multimeter across CAN-H/CAN-L (unpowered): ~120 Ω = fitted.
3. Check OBD pinout continuity, label the leads.
4. Set the buck to 5.0 V on the bench PSU (current limit low) before it ever meets an ESP32.
