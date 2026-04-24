<!-- consult selectively — grep, never read in full -->

# history.md

Chronological log of what was found and decided, newest first. Append-only. For *audit trails* — timing + rationale. For timeless "how does X work" knowledge, write into `notes.md` instead.

---

## 2026-04-25 — RS-232 pinout terminology and history (reference)

Logged as facts to remember when reading old industrial docs vs modern PC docs. (Strictly this is timeless knowledge that belongs in `notes.md`, but Ruben asked for it here as a learning record — kept dated for the audit trail.)

**The thing called "RS-232" standardises signals, not connector pinouts.** EIA RS-232 (1960) defines voltage levels (±3 V to ±15 V), bit timing, and a list of named signals (TxD, RxD, GND, DTR, DSR, RTS, CTS, DCD, RI). It also originally specified a **DB-25 connector with a specific pin assignment** (GND on pin 7, TxD on pin 2, RxD on pin 3, DTR on pin 20, DSR on pin 6, RTS on pin 4, CTS on pin 5, DCD on pin 8). For ~25 years, that DB-25 *was* the RS-232 pinout — the only one.

**There has never been an "official" RS-232 DB-9 pinout in the EIA RS-232 spec itself.** The DB-9 form factor on serial ports is purely a 1984 IBM PC/AT design choice — IBM dropped the 25-pin port for board-space reasons and reassigned the pins on the new 9-pin shell:

| Signal | IBM PC/AT DB-9 (1984)  |
|--------|-----------------------|
| DCD    | 1                     |
| RxD    | 2                     |
| TxD    | 3                     |
| DTR    | 4                     |
| **GND**| **5**                 |
| DSR    | 6                     |
| RTS    | 7                     |
| CTS    | 8                     |
| RI     | 9                     |

This IBM layout was later formalised — but only in 1990, by the TIA, not the EIA — as **TIA-574** ("9-Position Non-Synchronous Interface Between DTE and DCE", also written **EIA/TIA-574** or **EIA-574**). So:

- **DE-9** / **D-sub 9** = the *connector shape* (D-shaped shell, 9 pins, "E" body size in the D-sub family). Says nothing about which pin carries which signal.
- **TIA-574** = the *modern PC pinout on a DE-9*. This is what every USB-to-RS-232 adapter, every Windows COM port, every modem, every consumer DB-9 cable assumes today.
- **"RS-232"** by itself = ambiguous. At the signal level it's well-defined; at the connector level it could mean DB-25 (original) or TIA-574 (post-IBM).

**Why Fagor's 1990s-era CNC has a "weird" pinout.** Fagor put DB-9 connectors on their CNCs but kept the **EIA RS-232 DB-25 pin numbers** wherever they fit. Pin 7 = GND because that's what RS-232 *originally* specified; pin 6 = DSR same reason; pin 5 = CTS same reason; DTR didn't have a pin under 9 (it was originally pin 20) so it ended up on pin 9. So Fagor's pinout is **"the original RS-232 DB-25 layout compressed onto a DB-9 shell"** — entirely standards-compliant for the standard that existed when the 8050/8055 was designed. They just predate TIA-574 winning.

**The phrase to use when disambiguating**: "The CNC follows the original **EIA RS-232 DB-25** pin numbers preserved on a DB-9 shell. The PC follows **TIA-574** (the IBM PC/AT DB-9 layout). Both are 'RS-232' at the signal level."

**Other industrial gear with the same kind of legacy DB-9 pinout** (so this isn't a Fagor quirk):
- Heidenhain TNC controls (own DB-9 layout)
- Siemens Sinumerik older variants (DB-15)
- Allen-Bradley PanelView (own DB-9 layout)
- Cisco router console ports (RJ-45 with proprietary pinout, for years)
- Apple Mac serial ports 1984–1998 (Mini-DIN-8)

Every one of these requires a custom adapter cable to talk to a TIA-574 PC. **"RS-232 cable" without a pinout drawing is about as specific as "USB cable" — physically similar parts, not electrically interchangeable.**

**Practical implication for this repo**: when ordering a "Fagor 8055 DNC cable" or building one, the spec is "DB-9 male PC end (TIA-574) to DB-9 male CNC end (legacy EIA RS-232 DB-25-on-DB-9)" with the pin remap and handshake loopbacks already documented in `docs/rs232_pinout.md`.

---

## 2026-04-25 — Second open issue: the USB adapter + cable path may also be wrong

**What happened:** Ruben mentioned the peer is connecting via "a normal USB-C to RS-232 adapter from Amazon" plus (presumably) a generic DB-9 cable between the adapter and the cabinet's Harting hood.

**Why this matters:** commodity USB-to-RS-232 adapters expose the **standard** DB-9 DTE pinout (pin 5 = GND, pin 4 = DTR, pin 8 = CTS). The Fagor 8050/8055 CNC-side DB-9 is **non-standard** — pin 5 = CTS, pin 6 = DSR, pin 7 = GND, pin 9 = DTR (see `docs/rs232_pinout.md`). If the peer's cable between the adapter and the Harting is a generic straight-through or a generic null-modem cable, then **PC ground (pin 5) connects to CNC pin 5 (CTS)** and CNC ground (pin 7) is unterminated — which should give zero bytes through.

But bytes *are* going through: the `Bloques transmitidos` counter advanced during the failed COPY attempt. Two plausible explanations:

1. **The cabinet Harting hood contains a Fagor adaptation pigtail** that internally remaps PC DB-9 pin 5 → CNC DB-9 pin 7 and sets up the handshake loopbacks. This is what a proper factory install would look like on a 1998 machine. Opens-the-hood-and-looks resolves it.
2. **Chassis/mains ground is bonding the laptop and machine enough that RS-232 partially works without a proper signal-ground wire.** This would explain intermittent success, poor noise margin, and the "half-transferred" behaviour. Not sustainable — noise kills it the moment anything heavy switches in the shop.

**Open question for the technician visit**: have them open the Harting hood and tell us whether there's Fagor-specific pin remapping inside or whether the cable is wired as a generic DB-9. If generic, the cable needs replacing (easy: build a 3-wire cable per `docs/rs232_pinout.md`, or buy a Fagor-specific pre-wired cable like "CNC-SW-25M" on eBay).

**How to apply:** the "inspect Harting hood" task in `.agents/tasks.md` just got more important — it's no longer a formality for an audit trail, it's an active suspect. Even if `1001.pim` is replaced with a clean program, a marginal cable will keep producing intermittent failures. Both issues need to be resolved to have a reliable DNC path.

**Caveat for the minimal-test-program plan**: if the cable is marginal-but-coupling-via-chassis-ground, the 4-line test program might still get through (small payload, fewer chances for a bit to flip), and we'd incorrectly conclude the wire is fine. Safer: run the minimal test multiple times, *and* ask the technician to validate the wiring independently.

---

## 2026-04-25 — Retrospective: why did the first three debug passes yield no useful data?

**The chain of misdiagnosis.** Across the first three debug passes (repo scaffolding → "red text" symptom triage → forum survey) we progressively narrowed suspects to: dead battery → baud mismatch → flow control → cable pinout → USB-serial adapter → `P22` port routing. Each was plausible in isolation. None of them was the bug. The actual cause (file content 5–7× too big for the CNC's EEPROM and containing block numbers above the 8055-M's ceiling) was visible in:

1. The **`Bloques transmitidos` counter** on the CNC during the COPY attempt — that counter advancing *at all* means the wire is passing bytes, which should have moved "wire problem" off the suspect list on day one.
2. The **`INSTRUCCION INCORRECTA`** error at the end — an explicit "I parsed a line and it's not valid Fagor G-code" signal from the CNC. That message on its own rules out physical-layer problems and narrows to content/protocol. It was not a handshake or framing error.
3. The **"10 programas, 108669 bytes libres EEPROM"** status line on the UTILIDADES directory screen — announces the 108 KB memory ceiling. Given a 527 KB (post-junk-strip) source file, the ceiling alone is disqualifying.
4. The **`;51 errores, 0 warnings`** comment and the **9488 occurrences of `;Bloque CNC8025 incorrecto`** inside `1001.pim`. The file self-announces as a partially-failed 8025→8055 conversion.

None of these signals made it into the debug loop for two conversational rounds. **Why.**

- **Symptom framing drift.** "Red text in WinDNC" became the working description of the bug. We (me and the user) kept treating that phrase as a wire-layer fault because WinDNC's documentation says red = "port not initialised / peer not responding". But in the Fagor protocol, a CNC that NAKs an incoming block counts as "peer not responding" too — so red text has two disjoint root-cause classes (wire-layer *or* content-layer), and we only ever chased the first one.
- **PC-centric vs CNC-centric debugging.** Most of our time was spent on the laptop side: WinDNC config, COM port, USB-serial adapter, cable pinout. The peer was presumably operating the CNC in parallel, but we did not ask for CNC-side observations (screen contents, error numbers, memory state, what button was pressed) until the photo dump arrived this session. Every one of the four signals above was only visible on the CNC, not in WinDNC.
- **No minimal reproduction.** We never asked for a 4-line test program to be sent first. A minimal program would have either succeeded (proving the wire + file-content boundary lives in the file, not in the cable) or failed (proving the wire) in under a minute. Instead we reasoned top-down about a complex stack.
- **`1001.pim` committed without inspection.** The file appeared in Ruben's Downloads and got committed in the previous session *without me opening it*. A `head -5 1001.pim` at that moment would have shown `;51 errores, 0 warnings` — the bug's self-diagnosis was in the file the whole time.
- **Mirror confirmation on the CNC was missing for too long.** We had the `PARAM. LINEA SERIE 2` photo early (baud/parity/etc. all correct), but no photo of an *attempted transfer in progress* until this session. A stalled `Bloques transmitidos: 0` would have reopened wire-layer suspects; an advancing counter would have closed them. Either way, informative.

**Why "we got no data" on previous rounds.** We did get data — we just got the wrong data. We got WinDNC colour-coding (one bit: red or not red) and self-reported parameter values. What we didn't get was (a) what the CNC was doing while the laptop was red, (b) what file was being sent, (c) how big the CNC's available memory was, (d) what error, if any, the CNC showed. All four would have been a single on-site photo per question. Without them, every subsequent round was reasoning in a vacuum.

**What to do differently next time** (also now encoded as procedure in `.agents/notes.md` and as tasks):

1. **Before suspecting the wire, inspect the payload.** `file`, `head`, `tail`, `wc -l`, `grep -c` on the program file being sent. One minute of work.
2. **Before theorising, ask for a minimal reproduction.** A 4-line test program either succeeds (narrows to file content) or fails (narrows to wire). This should be the first physical action on any new DNC issue.
3. **Ask for the CNC's side of the story.** Every time WinDNC reports something, ask: "what does the CNC screen say? Is there an error number? How many bloques transmitidos?" The CNC is the authority on its own state; WinDNC is a thin proxy.
4. **Read committed artifacts.** When a user drops a file in the repo (as with `1001.pim`), open it and audit it, don't just list it in the commit.
5. **Write the bug description in CNC-native terms.** "INSTRUCCION INCORRECTA after N blocks" or "COPIAR aborted at bloque 52" — not "red text in WinDNC". The former points at the correct layer; the latter doesn't.

This entry is the meta-lesson for the repo; the concrete status for today's finding is in the next entry ("Port assignment confirmed ..." and "File `1001.pim` is the bug ...").

---

## 2026-04-25 — Port assignment confirmed on-screen; live reproduction captured

**What happened:** audit of Ruben's Downloads folder found 6 on-site photos from 2026-04-24 that hadn't made it into the repo yet — they document a live attempt at a DNC pull from the CNC's UTILIDADES screen. All six are now in `resources/photos/` (files prefixed `cnc_utilities_*` and `cnc_dnc_copy_*`) and the originals have been removed from Downloads per the "temp files don't live in Downloads" rule.

**Key finding — port assignment resolved:** `cnc_utilities_serie_selector.jpg` shows soft-keys labelled **`L. SERIE 1 (RS-422)`** and **`L. SERIE 2 (DNC)`**. So on *this* machine Line 2 is the RS-232 / DNC port (opposite of the typical 8055 OEM build the forum survey described). This **matches** the cabinet's "RS232 DNC" Harting label and the `DNC 2` indicator on the parameters screen. The `P22` routing concern is effectively settled — reading it is still worth an entry in the audit trail but it's no longer a high-priority suspect.

**Live reproduction captured:**
- `cnc_dnc_copy_prompt.jpg` — UTILIDADES → COPIAR → LINEA SERIE 2 (DNC) prompt, asking for a destination program number.
- `cnc_dnc_copy_transmitting.jpg` — "COPIAR LINEA SERIE 2 EN P999", `Bloques transmitidos: N` progress counter, `ABORTAR` soft-key visible. Some blocks *do* arrive.
- `cnc_dnc_copy_error.jpg` — post-failure: directory now shows 11 programs (P999 was created) but the status bar reads **`INSTRUCCION INCORRECTA`**.

**Why this changes the debug model:** the symptom is *not* a dead port — blocks are arriving and P999 was created. What the CNC is rejecting is the *content* of those blocks. "INSTRUCCION INCORRECTA" means the 8055 parsed a line it doesn't recognise as valid Fagor G-code. Likely causes (in rough order):

1. **File format issue on the PC side** — ESC, NUL, or other binary trailer that the PC-side sender left in; mixed/wrong line endings (CR-only vs LF-only vs CRLF); BOM at the start of the file; a non-Fagor header line.
2. **Baud-induced garbling** at 19200 — bytes get corrupted on the wire, one bit flipped produces an invalid opcode. This is consistent with the forum advice to drop to 9600.
3. **Flow-control mismatch causing mid-line drops** — blocks land with missing characters, producing invalid G-code.
4. **Encoding mismatch** — the 8055 expects 7-bit-clean ASCII; anything high-bit (accented chars in comments, smart quotes from a Word export) breaks parsing.

The `resources/programs/1001.pim` file committed in the previous session is probably the program the peer was trying to send. Inspected it — and **it's fundamentally broken for this CNC**, for two independent reasons:

1. **Size.** The file is 792 KB / 29,488 lines. After stripping the junk (see below), it's still 527 KB / 20,000 lines. The CNC shows `108669 bytes libres EEPROM` on the UTILIDADES screen — i.e. ~108 KB of free program memory. The file is **~5–7× too big to fit**. Partway through the transfer the CNC runs out of room and aborts.
2. **Block numbering.** Block numbers in the file climb to **N99995** (sample: `N99950`, `N99955`, `N99960` … `N99995`). The 8055-M's block-number ceiling is **N9999** — everything above that is a syntax error. That alone would produce `INSTRUCCION INCORRECTA`.
3. **Provenance.** The header is `%1001.PIM,M,` (valid 8055-M), but inside there are **9488** occurrences of `;Bloque CNC8025 incorrecto` (converter-annotation comments), plus a one-liner `;51 errores, 0 warnings` near the top. The file was clearly produced by a converter that translated a **Fagor 8025** program (earlier generation) to 8055 format and flagged 51 block types it couldn't convert — leaving the bad blocks plus redundant comment markers in place.

So the root cause of the failed transfer is the file, not the wire. DNC transport is demonstrably passing bytes (blocks were transmitted, P999 got created). The CNC is legitimately rejecting the content.

**Fix path:** either (a) re-generate a clean 8055-M program from the original source on the PC's CAM side (don't go through the 8025→8055 converter), or (b) use **DNC execution mode** instead of COPY — the 8055-M can run large programs by streaming them from the PC block-by-block without storing them in EEPROM, which works around the 108 KB limit. That's also chapter material in `fagor_8055m_operating.pdf`. Either way, don't chase the cable / adapter / baud any further until a known-good program is loaded; the transport is fine.

**How to apply:** next on-site session, run the existing "drop baud to 9600" task (procedure already in `tasks.md`) *and* check the content of the program file before sending. If 9600 fixes it, it was baud-induced garbling. If it still fails with INSTRUCCION INCORRECTA, the program file itself is the bug and we strip it down to a minimal test (a single-line `%` header + `(MSG "test") M30`) to isolate format vs. transport. Don't change the cable or adapter yet — the port is demonstrably passing bytes.

Current-symptom section in `AGENTS.md` updated to reflect this (P22 deprioritised; new #5 step about inspecting the sent file).

---

## 2026-04-24 — Forum survey: WinDNC "red text" is a documented port-health indicator, not a bug

**What happened:** a subagent surveyed cnczone, industryarena, practicalmachinist, foro.metalaficion, usinages.com, aggsoft, atrump, and the WinDNC manual itself to see how common this symptom is and what actually fixes it. Ranked list and known-working recipe went into `.agents/notes.md` ("WinDNC red text / red background — what it really means and the known fixes").

**What was surprising:**

1. **The red colour is documented.** Fagor's own WinDNC manual calls it out: white = port initialised OK, red = port not initialised / peer not responding. That's why almost nobody on forums describes it as "red text" — they describe the underlying condition ("no ACK", "error de transmisión"). So we shouldn't chase "red text" as a specific bug; it's just the port-layer-unhealthy indicator.
2. **Parameter `P22` is the smoking gun candidate.** On most 8055 OEM builds, `X3` = RS-232 (9-pin), `X4` = RS-422 (25-pin), and `P22` selects which physical port carries DNC (`P22=1` forces RS-232 + Ethernet, other values route DNC to RS-422). The Harting hood on the cabinet is labelled "RS232 DNC" but the CNC screen puts DNC on Line 2. If `P22` is set such that Line 2 lands on X4/RS-422 but the cable is plugged into X3/RS-232, you get exactly "half-working" comms from stray differential levels. **Read `P22` next visit — it's the most suspicious single item on the list.**
3. **Our 19200 baud is on the edge.** Early 8055s can't run 19200 reliably; the canonical debug move is drop to 9600 first, prove it works, raise later. Worth trying before any cable work.
4. **Dead battery will cause "fixes that don't stick".** Multiple threads report parameters reverting on next power-cycle when the CPU-board lithium is dead. Ours is confirmed dead (frozen clock). **Replace the battery before spending long on the bug** — otherwise any serial-param fix evaporates at the next boot.
5. **"Red text" in WinDNC specifically is often a corrupted WinDNC install.** At least one thread fixed it with a plain reinstall. Cheap to try early.
6. **The converged "works" recipe is simple:** FTDI FT232 adapter (not Prolific), DB9F-DB9F cable with `2↔3, 3↔2, 5↔5`, `4-6-8` shorted at *each* hood, CNC at 9600/8/N/1/Fagor/XON-XOFF, WinDNC matching, hardware handshake off on the PC side.

**How to apply:** next on-site session, work this order (each step validates the previous):
(a) Reinstall WinDNC (cheap), (b) replace the CPU-board battery + backup/restore params, (c) read and note `P22`, (d) drop baud to 9600 on both ends, (e) if still broken, open the Harting hood and verify the cable pinout (4-6-8 jumpered each end), (f) if still broken, swap the USB-serial adapter for an FTDI. Log each step's outcome under `logs/YYYY-MM-DD_.../notes.md`.

**Sources:** cnczone [138526](https://www.cnczone.com/forums/fagor-automation/138526-pin-rs232-fagor8050-55-a.html) [166014](https://www.cnczone.com/forums/fagor-automation/166014-connect-pc-fagor-8055-a.html) [233210](https://www.cnczone.com/forums/fagor-automation/233210-cnc.html); practicalmachinist [battery-thread](https://www.practicalmachinist.com/forum/threads/lost-parameters-due-to-backup-battery-failure-fagor-8055.345308/) [800T-thread](https://www.practicalmachinist.com/forum/threads/fagor-800t-based-controller-unable-to-connect-through-rs232-port.401955/); aggsoft [DNC setup guide](https://www.aggsoft.com/cnc-dnc/fagor-8040-8050.htm); atrump [RS-232 setup PDF](https://atrump.com/images/tech_support/04%20Control/Fagor/RS%20232%20setup%20and%20communication.pdf); foro.metalaficion [2333](https://foro.metalaficion.com/index.php?topic=2333.0) [5682](https://foro.metalaficion.com/index.php?topic=5682.0); usinages.com [95854](https://www.usinages.com/threads/probleme-connexion-fagor-8055-mc-pc-par-rs232.95854/); WinDNC manual [es.scribd](https://es.scribd.com/doc/129078120/WinDNC).

---

## 2026-04-24 — Control confirmed as 8055-M; 8055-M manual set pulled

**What happened:** my earlier photo-read of the boot screen (`cnc_boot_screen.jpg`) called the control "Fagor 8050 SERCOS" — on that CRT, `0` and `5` are visually ambiguous. Ruben confirmed on-site that the control is actually **Fagor 8055-M** (milling variant, as opposed to -T for lathes). Full photo-verification via the `UTILIDADES → VERSION` screen still pending but the overall identification is considered settled.

A second subagent hunted for 8055-M documentation online and returned a clean URL set on the DMS CNC Routers and Endo.com.tr mirrors (both login-free). Downloaded **eight PDFs** into `resources/manuals/` (Ruben also supplied the Spanish-edition Operating manual from his own download). After first gitignoring the large ones per the earlier "a few MB" policy, Ruben overrode that — **all PDFs are now committed**; total is ~26 MB which is well within GitHub limits and means a fresh clone gives the full documentation set offline. The PDF policy in `AGENTS.md` and `docs/README.md` is updated to match.

Committed PDFs: `fagor_8050_rs232_setup.pdf`, `fagor_8055_installation.pdf`, `fagor_8055m_operating.pdf` (EN), `fagor_8055m_operating_es.pdf` (ES), `fagor_8055m_programming.pdf`, `fagor_8055m_errors.pdf`, `fagor_8055m_examples.pdf`, `fagor_8055i_ordering_handbook.pdf`.

**Why this matters:** the 8055 family ships docs as **separate books** (Operating / Installation / Programming / Errors / Examples) — different from the 8050's "two big books" layout. All the 8050 URLs we'd gathered earlier were for the wrong generation's doc structure. We kept the 8050 links as a fallback in `docs/README.md` in case the final photo-verification flips things back, but the 8055-M set is now authoritative.

Also relevant: 8055 (rack) and 8055i (integrated single-board, ~2003) share the same manual set — `·M·` is the software kernel, not a hardware variant. So either hardware edition is fine to work from.

**How to apply:** for any serial / DNC / pinout debugging, open `resources/manuals/fagor_8055_installation.pdf` first — it's the authoritative reference. The WinDNC chapter in `fagor_8055m_operating.pdf` covers the PC-side configuration dialog. If Ruben wants to re-fetch the gitignored PDFs from a fresh clone, the URLs are in the "Local-only" table of `docs/README.md`.

---

## 2026-04-24 — Flow-control mismatch identified on WinDNC side

**What happened:** working through the `PARAM. LINEA SERIE 2` screen photo on the CNC (photo taken on-site, shown in chat 2026-04-24), we compared the CNC-side serial parameters against WinDNC's config on the Windows laptop. **Flow control was set differently on the two ends** — the CNC has `P010 XONXOFF = ON`, but WinDNC was not configured to match. This is a plausible root cause for the red-text / no-ACK symptom: when the receiver fills its buffer it sends XOFF, and if the PC isn't watching for it (because it's expecting hardware RTS/CTS or "none") bytes get dropped, the Fagor protocol NAKs, and WinDNC shows red.

**Why this matters:** narrowing the suspect list. Until today, suspects were (a) DNC not enabled, (b) wrong `PROTOCOL` param, (c) baud mismatch, (d) non-standard null-modem pinout, (e) flaky USB-serial adapter. Photo now **rules out (a)–(c)**: `PWONDNC=YES`, `PROTOCOL=1` (Fagor), `BAUDRATE=008` = 19200 bps, `NBITSCHR=1` = 8 bits, `PARITY=0` = none, `STOPBITS=0` = 1. The flow-control finding promotes that to the top of the list, with (d) cable pinout second and (e) adapter third.

**How to apply:** next on-site session, set WinDNC flow control to **XON/XOFF** (not Hardware, not None) to match `P010 = ON`. Before changing anything else, press **F6 SALVAR** on the CNC parameters screen to back up current values. Then flip `P006 DNCDEBUG = YES` temporarily and capture the CNC's trace during the next WinDNC attempt.

---

## 2026-04-24 — Machine identity corrected; on-site photo pack received

**What happened:** 6 photos from the machine arrived via `~/Downloads/Photos-3-001 (2).zip`. Nameplate and boot-screen readings force two corrections to the repo's original scaffolding:

- Control is **Fagor 8050 SERCOS**, not 8055 as originally assumed. (Submodel — M / T / MC / TC — still TBD; panel rear label needs a photo.)
- Machine is **Enrique Holke F 2230**, Nº 997, Esquema Nº 011001, built 1998, 8000 kg, 400 V / 50 Hz. Installed at **ONCE UTT Madrid**.
- Machine is a **vertical machining center**, not a knee-mill (sibling models F2233 / F2242). The Tony Griffiths data pack HM684 on lathes.co.uk covers the older Holke knee-mill line and is the **wrong generation** — do not buy it.
- CRT clock is frozen at `Viernes 01 Julio 2022 02:22:41` → CPU-board lithium battery is dead. Replace before parameter memory follows.

**Why this matters:** the Fagor 8050 doc set is split differently from the 8055 (one **OEM manual** covering install + SERCOS + params + errors, one **USER manual** per submodel covering operating + programming). Some of the earlier links and parameter names we'd have used for an 8055 don't map 1-to-1. Also: calling this a VMC not a knee-mill changes which Holke lineage docs are relevant.

**How to apply:** all documentation searches must specify 8050 (not 8055); machine is VMC (not knee-mill); prefer mirrors (manualzz) over Fagor's own site (which has dropped the 8050 entirely).

---

## 2026-04-24 — Research round: manual sources located

**What happened:** three background subagents ran in parallel on Fagor, Holke, and INTZA docs. Consolidated findings (URLs + contacts + caveats) went into `docs/README.md`. Key outcomes:

- **Fagor:** 8050 is fully EOL at Fagor; no active support, no download page. Community mirror = manualzz.com. Single most valuable doc = `fagor_8050_rs232_setup.pdf` (220 KB Fagor app note, now committed in `docs/`) — covers the exact `PARAM. LINEA SERIE 2` screen we have a photo of, plus the Fagor-specific null-modem pinout. For everything else, the OEM manual (~5–8 MB, gitignore-scale) is the priority.
- **Holke:** manufacturer is **still in business** — `comercial@eholke.com`, +34 943 310 744 (same number as the 1998 nameplate). They are almost certainly the only source for Esquema Nº 011001 (does not appear online). Maintenance arm = SEMAVE (division of Enrique Holke S.L.). Next step: email them in Spanish with the nameplate photo attached.
- **INTZA:** manufacturer also still active (`intza@intza.com`, +34 943 852 600). Current photo's nameplate (TIPO, Nº) is not legible — need a **re-shoot** before contacting them.

**Why this matters:** Holke still existing at the same phone number is a very strong signal that the factory archive for this machine survives. Esquema Nº 011001 becomes reachable via one email instead of a forum dig. INTZA is blocked on a photo re-shoot.

**How to apply:** tasks for Ruben (email Holke, re-shoot INTZA plate, then email INTZA, fetch the Fagor OEM manual) are in `.agents/tasks.md`. Don't burn time searching forums for Holke F 2230 schematics — go direct.
