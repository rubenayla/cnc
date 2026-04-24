<!-- consult selectively — grep, never read in full -->

# notes.md

Timeless reference knowledge about this machine — *how things work*, not *what happened when*. For dated events use `.agents/history.md`. For project instructions use `AGENTS.md`.

Organized by topic. Grep for a keyword; don't read top-to-bottom.

---

## Fagor 8055-M serial line — PARAM. LINEA SERIE 2

The machine's DNC port is assigned to **Serial Line 2** on the CNC (`DNC 2` indicator, top-right of the parameters screen). The parameter screen is reached via F7 → setup → machine parameters → serial lines. Layout of parameters P000–P010:

| # | Name | Typical / current | Meaning |
|---|------|-------------------|---------|
| P000 | **BAUDRATE** | `008` = 19200 bps | Index into a table: 0=110, 1=150, 2=300, 3=600, 4=1200, 5=2400, 6=4800, 7=9600, 8=19200. Must match PC exactly. |
| P001 | **NBITSCHR** | `1` = 8 data bits | `0` = 7 data bits, `1` = 8. **Always leave at 1** for PC DNC — 7-bit drops the top bit of every byte and destroys the Fagor protocol's block-check. Mixed 7/8 is the single most common garbled-characters cause. |
| P002 | **PARITY** | `0` = none | `0` none, `1` odd, `2` even. |
| P003 | **STOPBITS** | `0` = 1 stop bit | `0` 1 stop bit, `1` 2 stop bits. |
| P004 | **PROTOCOL** | `1` = Fagor DNC | `0` raw / no protocol (for some third-party apps), `1` Fagor block-checked protocol with ACK/NAK. **WinDNC needs `1`.** |
| P005 | **PWONDNC** | `YES` | Enable DNC automatically at power-on. YES = no need to toggle DNC on each boot. |
| P006 | **DNCDEBUG** | `NO` | DNC trace logging on the CNC side. Flip to `YES` temporarily when debugging; read the trace after reproducing the fault. |
| P007 | **ABORTCHR** | `0` | ASCII code of the "abort transmission" character. `0` = disabled / default. |
| P008 | **EOLCHR** | `0` | End-of-line character code. `0` = default LF. |
| P009 | **EOFCHR** | `0` | End-of-file character code. `0` = default. |
| P010 | **XONXOFF** | `ON` | Software flow control. See "Flow control" below. |

### Soft-keys on this screen

- **F1 EDITAR** / **F2 MODIFICAR** — edit the highlighted parameter (label wording differs slightly across firmware revisions)
- **F3 BUSCAR** — search by parameter number
- **F4 INICIALIZ.** — reset parameters to factory defaults (destructive; never press without saving first)
- **F5 CARGAR** — load parameters from external storage
- **F6 SALVAR** — **save parameters to external storage. Always do this before changing anything.**
- **CAP INS** — screen indicator only, not a soft-key (caps-lock / insert state of the alphanumeric keypad)

### Canonical WinDNC ↔ CNC settings

For this machine, match the PC to the CNC screen above:

| WinDNC setting | Value |
|----------------|-------|
| Baud           | 19200 |
| Data bits      | 8 |
| Parity         | None |
| Stop bits      | 1 |
| Protocol       | Fagor |
| Flow control   | XON/XOFF |
| COM port       | whichever COM the USB-serial adapter enumerates as |

Any single mismatch = red text in WinDNC.

---

## Serial flow control, applied to the 8055-M

**The problem it solves:** the sender has no idea how fast the receiver empties its input buffer. Overflow silently drops bytes. Flow control is the "pause / resume" backchannel.

Three options a serial port can pick between:

- **None** — no signalling; overflows silently.
- **XON / XOFF — software.** Receiver sends ASCII 19 (XOFF) to say "stop" and ASCII 17 (XON) to say "go". Works over a 3-wire cable (TX / RX / GND). Safe for text and G-code, unsafe for binary (byte values 17 and 19 collide). This is the default for DNC.
- **RTS / CTS — hardware.** Two extra wires (RTS output, CTS input) in the DB connector signal "I'm ready / stop sending". Faster and safe for binary, but requires the cable to actually wire pins 7 and 8 through and both ends to be configured for it.

On the 8055-M specifically:

- `P010 XONXOFF = ON` enables software flow control on the CNC.
- The Fagor-standard DNC cable is **3-wire (TX, RX, GND) with pins 4-6-8 jumpered at each end** — i.e. hardware handshake is faked locally, not wired end-to-end. Setting WinDNC to "Hardware / RTS-CTS" will make the PC wait forever for a CTS that never comes.
- **Correct WinDNC setting: XON/XOFF.** Not "Hardware", not "None".

Debug heuristic:

- Data gets through in small chunks then stalls → flow-control mismatch.
- Nothing gets through or all garbage → baud / bits / parity / stop bits mismatch.
- Small files work, big files fail at a predictable point → flow control missing entirely.

---

## Fagor 8055-M — family and docs

- **8055** (rack, ~1999) and **8055i** (integrated single-board, ~2003) share the same manual set and the same `·M·` milling software kernel. Fagor titles them "CNC 8055 / CNC 8055i" regardless of hardware variant.
- Suffixes: `·M·` = milling, `·T·` = turning, `·MC·` / `·TC·` = conversational (forms-driven UI layered on top of M/T respectively). This machine is `·M·`.
- Docs ship as **separate books**: Operating (+ WinDNC chapter), Installation (OEM — machine params, pinouts), Programming (G-code + cycles), Error Solving, Examples, Ordering Handbook. The Installation manual is generic to M and T; other books are milling-specific.
- Predecessor was the **8050** (earlier-gen, different doc layout: two big books only). Some early searches in this repo returned 8050 mirrors by mistake — when picking manuals, specify 8055.
- Current Fagor line is **8058 / 8060 / 8065 / 8070** ("CNCelite"). The 8055 family is supported but no longer under active feature development. WinDNC is still distributed for 8055; new machines use DNC over Ethernet with "CNC Explorer".

---

## Machine layout — Holke F 2230

- **Vertical machining center** (not a knee-mill). Siblings F2233, F2242.
- Fagor 8055-M CNC, 1200×500 mm table, X/Y/Z ≈ 1000/500/620 mm, ISO 40 spindle, ~17 kW.
- Nameplate: 8000 kg (includes pallet / coolant tank); bare machine spec is ~6000 kg.
- Manufacturer **Enrique Holke S.L.U.** still exists — same phone (+34 943 310 744) as the nameplate, now in Arroa-Zestoa (Gipuzkoa). Sales `comercial@eholke.com`, maintenance arm **SEMAVE**. They are the only realistic source for Esquema Nº 011001.
- Lubrication: **INTZA** centralized single-line oil lubricator, ~30 bar. Manufacturer `intza@intza.com`, +34 943 852 600, still active.

---

## DNC cabling — where it plugs in

- Cabinet side: Harting HAN hood labelled **"RS232 DNC"** on the electrical enclosure (see `resources/photos/rs232_dnc_connector.jpg`). Actual DB connector inside the hood hasn't been opened/photographed yet.
- CNC side: DNC is assigned to **Serial Line 2** per the parameters screen (`DNC 2` header).
- **On most 8055 OEM builds the ports map like this**: `X3` = RS-232 (9-pin male SUB-D), `X4` = RS-422 (25-pin male). Parameter **`P22`** selects which physical port carries DNC — `P22=1` forces RS-232 + Ethernet, RS-422 disabled; other values route DNC to RS-422. The Harting hood is labelled "RS232" but the screen puts DNC on Line 2 — **the P22 value has to be read before trusting either label**. Mis-routed DNC (CNC listening on X4/RS-422, cable plugged into X3/RS-232) presents exactly as "half-working" comms: stray differential levels partially couple through the single-ended PC receiver.

---

## WinDNC "red text / red background" — what it really means and the known fixes

Based on a forum/blog survey 2026-04-24. Sources catalogued in `.agents/history.md` under the same date.

### What the colour means
Per Fagor's own WinDNC manual: white background = serial port initialised OK, **red background = port not initialised or peer not responding**. "Red text" is just the generic "port-layer not healthy" indicator — it isn't a specific bug name. On forums people describe the underlying condition instead ("no ACK", "error de transmisión", "won't read from serial port").

### Ranked fixes (most common first)

1. **Cable pinout — jumper 4-6-8 at *each* end.** Single most-repeated field fix. Canonical null-modem for modern 8055: `2↔3, 3↔2, 5↔5`, with pins **4-6-8 shorted together at each DB9 hood**. If 4-6-8 isn't bridged locally, the CNC's own DTR/DSR/CTS loop fails and it sits silent — exactly the "PC sends, CNC never ACKs" pattern. Older EPROM-era 8055s want `2↔2, 3↔3, PC-5↔CNC-7`, with `4-6-8` at PC and `5-6-9` at CNC.
2. **Drop baud to 9600 first.** Early 8055s cannot reliably run 19200; some tops out at `P000=7` (9600). Our machine is at `P000=8` (19200) which is on the edge. Debug recipe: drop to 9600 on both ends, prove it works, then raise.
3. **Flow control mismatch** — XON/XOFF must be ON on both ends, with hardware/RTS-CTS/DSR/DTR all **off**. WinDNC profile tweaks: `Use DTR = False`, `Use RTS = False`, `Require DSR = False`, `Require CTS = False`, `Xon=17`, `Xoff=19`. We already confirmed `P010 XONXOFF = ON` on the CNC — PC side must match.
4. **USB-serial adapter: FTDI FT232, not Prolific PL-2303.** Prolific's newer drivers brick cloned chips and ~all the cheap eBay/Amazon cables are clones. The community has converged on FTDI; Fagor-specific pre-wired FTDI cables sell on eBay as "CNC-SW-25M".
5. **P22 wrong → CNC listening on RS-422 port.** See the cabling section above. High-suspicion item for this machine given the Harting label / LINE 2 mismatch.
6. **File EOF character.** WinDNC expects NUL-terminated files; programs round-tripped through a generic terminal can pick up an ESC trailer that hangs the handshake. `P004=1` is correct for us, but file contents matter too.
7. **WinDNC reinstall** — occasional reports of a corrupted install presenting as persistent red. Cheap to try.
8. **Ground loops.** Old laptop on battery works; grounded desktop doesn't. USB-serial adapter (galvanic break) as workaround, or run the laptop unplugged.
9. **Dead backup battery → parameters silently revert.** 3.6 V Saft LS14500 on the CPU board. Ours is confirmed dead (frozen clock). Common failure mode: "fixed yesterday, broken today." Any serial-params fix will not stick until the battery is replaced and a backup/restore is done. **Replace battery before spending long on the bug.**

### Known-working consensus recipe

- FTDI FT232-based USB-serial adapter (not Prolific)
- DB9F-DB9F cable: `2↔3, 3↔2, 5↔5`, and `4-6-8` shorted at *each* hood
- CNC params: `P004=1` (Fagor protocol), `P001=1` (8 bits), `P002=0` (no parity), `P003=0` (1 stop), `P010=ON` (XON/XOFF), **`P000=7`** (9600 bps first, raise to 19200 only after it's proven stable)
- WinDNC profile: same baud + 8/N/1 + XON/XOFF, hardware handshake disabled
- Verify `P22` value and that the cable physically lands on the matching port (X3 for RS-232)

---

## CRT clock frozen → dead battery

The boot screen showing a fixed date (`01 Julio 2022`) means the CPU-board backup lithium is dead. On the 8055-M this is a standard failure — replace the battery before parameter memory follows. Identify battery type from the CPU board itself (usually a small soldered lithium or a coin-cell holder); don't guess — open and look.
