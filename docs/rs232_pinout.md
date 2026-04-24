<!-- reference — read when relevant -->

# RS-232 pinout — Fagor 8050 / 8055 CNC ↔ PC

Transcribed from Fagor's own app note `resources/manuals/fagor_8050_rs232_setup.pdf` (scroll to page 2, "Serial Port Cable Connection CNC - PC"). The 8055-M uses the same pinout unless the installation manual `resources/manuals/fagor_8055_installation.pdf` says otherwise for a specific hardware revision — **verify before wiring**.

## CNC side (9-pin DB-9, at the back of the operator panel / inside the cabinet Harting hood)

Important: **this is NOT the standard DB-9 pinout.** Fagor reassigned several pins. On a generic DB-9 you expect pin 5 = GND, pin 4 = DTR, pin 7 = RTS, pin 8 = CTS. Fagor does not:

| CNC pin | Signal | Direction (from CNC's point of view) |
|---------|--------|---------------------------------------|
| **2**   | **TxD** | Out — data from CNC to PC            |
| **3**   | **RxD** | In — data from PC to CNC             |
| **5**   | CTS    | In — "PC is ready to receive"         |
| **6**   | DSR    | In — "PC is powered on"               |
| **7**   | **GND** | Signal ground (non-standard — usually pin 5 elsewhere) |
| **9**   | DTR    | Out — "CNC is powered on"             |

Pins 1, 4, 8 are unused.

## PC side (9-pin DB-9 — standard, as it ships from the PC)

| PC pin | Signal |
|--------|--------|
| 2      | RxD   |
| 3      | TxD   |
| 4      | DTR   |
| 5      | GND   |
| 6      | DSR   |
| 8      | CTS   |

Pin 1 (DCD), 7 (RTS), 9 (RI) unused for this application.

## PC side (25-pin DB-25 — legacy, same signals)

| PC pin | Signal |
|--------|--------|
| 2      | TxD   |
| 3      | RxD   |
| 4      | CTS   |
| 6      | DSR   |
| 7      | GND   |
| 20     | DTR   |

## Cable diagram — CNC (DB-9) ↔ PC (DB-9)

Three wires cross between the two ends; handshake lines are **looped back locally** on each end so neither side ever has to wait for a handshake signal that doesn't arrive.

```
  CNC side (DB-9M)                      PC side (DB-9F)
  ─────────────────                      ───────────────
  2  TxD  ──────────────────────────────  2  RxD
  3  RxD  ──────────────────────────────  3  TxD
  7  GND  ──────────────────────────────  5  GND
                                          
  5  CTS ┐                               4  DTR ┐
  6  DSR ├── jumpered (shorted)          6  DSR ├── jumpered (shorted)
  9  DTR ┘                               8  CTS ┘
```

- 3 wires between ends: `TxD ↔ RxD`, `RxD ↔ TxD`, `GND ↔ GND`
- On the **CNC hood**, solder pins 5, 6, 9 together (CTS/DSR/DTR loopback)
- On the **PC hood**, solder pins 4, 6, 8 together (DTR/DSR/CTS loopback)

## Cable diagram — CNC (DB-9) ↔ PC (DB-25, legacy)

Same logic, PC pins are different:

```
  CNC side (DB-9M)                      PC side (DB-25F)
  ─────────────────                      ────────────────
  2  TxD  ──────────────────────────────  3  RxD
  3  RxD  ──────────────────────────────  2  TxD
  7  GND  ──────────────────────────────  7  GND
                                         
  5  CTS ┐                               4  CTS ┐
  6  DSR ├── jumpered (shorted)          6  DSR ├── jumpered (shorted)
  9  DTR ┘                               20 DTR ┘
```

## Build notes

- **Cable length ≤ 15 m (49 ft).** Fagor's limit. Longer runs pick up noise; drop baud or use a shorter cable.
- **Shield connected at the CNC end only** (one-ended shield prevents ground loops). Don't tie the shield to GND on the PC end.
- **Don't route near power lines, transformers, or VFDs.** Standard shielded serial cable is not immune to the switching noise a machine-tool cabinet generates.
- Bought cables labelled "null-modem" or "Fagor compatible" almost certainly assume the *standard* DB-9 pinout, which is wrong for an 8050/8055. Either build it yourself or buy a cable from a Fagor reseller (e.g. "CNC-SW-25M" on eBay, FTDI-based, pre-wired for Fagor).

## Comparison with the forum-consensus "modern 8055" pinout

The forum-survey recipe (see `.agents/notes.md`) says `2↔3, 3↔2, 5↔5, and 4-6-8 shorted at each hood`. That's the **standard DB-9** interpretation and works on 8055i / later 8055 revisions where Fagor re-pinned to the standard DB-9 layout.

So there are effectively **two pinouts in the wild**, both called "Fagor 8055 RS-232":

1. **Legacy / 8050 / early 8055** — CNC pin 7 = GND, pin 9 = DTR, 5/6/9 jumpered. Documented in `fagor_8050_rs232_setup.pdf` — the version above.
2. **Later 8055 / 8055i** — CNC pin 5 = GND, pin 4 = DTR, 4/6/8 jumpered. Standard DB-9.

**If you don't know which your machine is:** open the cabinet Harting hood (task already in `.agents/tasks.md`) and read the DB connector's pin numbering, then confirm against `fagor_8055_installation.pdf`'s hardware chapter (the manual has diagrams per hardware revision). This machine is a 1998 build so it leans toward recipe #1; `cnc_utilities_serie_selector.jpg` showing "L. SERIE 1 (RS-422), L. SERIE 2 (DNC)" is consistent with either.

## For the WinDNC laptop specifically

- Don't chase the pinout further unless a **minimal test program** (`%0999\n(MSG "TEST")\nM30\n`, 4 lines) fails to transfer. A transfer that fails on `1001.pim` but works on the minimal program proves the cable is fine — see `.agents/history.md` 2026-04-25. Given the `Bloques transmitidos` counter advanced during the last attempt, the cable is very likely already correct.
