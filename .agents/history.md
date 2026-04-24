<!-- consult selectively — grep, never read in full -->

# history.md

Chronological log of what was found and decided, newest first. Append-only. For *audit trails* — timing + rationale. For timeless "how does X work" knowledge, write into `notes.md` instead.

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
