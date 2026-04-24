<!-- read in full — kept under 150 lines -->

# AGENTS.md — cnc

Fagor 8055-M SERCOS CNC troubleshooting repo — see `docs/machine.md` for the identification trail. Owner: Ruben. Machine owner: a friend (peer operates the Windows laptop on-site); the mill itself lives at ONCE UTT Madrid.

## What this repo is for
1. Collecting Fagor 8055 / WinDNC documentation in one place
2. Recording SSH + network details for the Windows laptop that drives the CNC
3. Logging debugging sessions (WinDNC logs, screenshots, what was tried, what the CNC did)
4. Becoming a durable reference if the kart team (or anyone else) inherits this machine later

## How to work here
- Read `.agents/tasks.md` before starting. It's the shared board between Ruben and AI agents. Claim a task by moving it to In Progress with date + agent ID; don't grab something already claimed.
- Each debugging session gets its own dated folder under `logs/` (e.g. `logs/2026-04-21_red_text/`) with the raw evidence and a `notes.md` summary. Keep evidence together so a single folder read reconstructs the session.
- Markdown notes live in `docs/`. Binary assets (vendor PDFs, photos) live in `resources/`. All current manuals are committed; source URLs are listed in `docs/README.md` for edition checks and re-fetching. If the manual pool grows past ~100 MB, revisit and move the biggest PDFs out-of-band or to git-lfs.
- Never commit secrets. Windows laptop password, API keys, anything sensitive → macOS Keychain or `~/.ssh/config`, not this repo. If a log file contains credentials, redact before committing.

## Conventions
- Dates in filenames and log entries: `YYYY-MM-DD`.
- SSH host alias for the Windows laptop (once set up): `cnc-laptop` in `~/.ssh/config`. CNC itself has no shell (embedded; FTP only, default IP seen on display: 172.16.1.1).
- Don't invent facts about the machine. If a parameter, IP, or pinout is unknown, mark it `TBD` in notes and add a task to find out.

## Known facts (as of 2026-04-24)
- Machine: **Enrique Holke F 2230**, Nº 997, year 1998, 8000 kg, 400 V / 50 Hz. Full details in `docs/machine.md`.
- Control: **Fagor 8055-M SERCOS** (Ruben's on-site read, 2026-04-24). "-M" = milling variant. Photo-confirmation of the VERSION screen / panel-rear label still pending. Firmware still TBD.
- Drive rack: APS-24 power supply + servo + spindle drives, SERCOS fiber ring between CNC and drives
- APS-24 has its own small keypad/display (PS-25B4 style) that plugs into the PS front, unrelated to PC comms
- DNC port is brought out to a Harting hood on the cabinet labelled **"RS232 DNC"** (see `resources/photos/rs232_dnc_connector.jpg`). Actual DB pinout inside the hood is TBD.
- CNC exposes FTP over Ethernet (default on 172.16.1.1 in the earlier screenshot); no SSH on the CNC itself
- Windows laptop has WSL Ubuntu installed but SSH will target Windows directly, not WSL (WSL can't see COM ports without usbipd-win plumbing)
- CRT clock is frozen (`01 Julio 2022`) → CPU-board backup battery is dead; schedule a replacement.

## Current symptom
WinDNC: when peer types commands, characters show against a **red background** — which Fagor's WinDNC manual documents as "serial port not initialised / peer not responding". It's a port-layer-unhealthy indicator, not a specific bug name.

**Parameters confirmed correct from the `PARAM. LINEA SERIE 2` photo** (19200 / 8 / N / 1 / Fagor protocol / DNC auto-on / XON-XOFF on). Debug order below is drawn from a forum survey (see `.agents/history.md` 2026-04-24 "Forum survey" entry and the fixes section of `.agents/notes.md`):

1. **Replace the CPU-board Saft LS14500 lithium first** — battery is dead (frozen clock). Any params fix will silently evaporate at next power-cycle until this is done.
2. **Reinstall WinDNC** on the laptop — cheap; occasional corrupted-install cause.
3. **Read `P22`** on the CNC — decides whether DNC rides on `X3` (RS-232) or `X4` (RS-422). If wrong, the cable is plugged into the wrong physical port.
4. **Drop baud to 9600** on both ends and retest. Early 8055s can't reliably do 19200; ours is `P000=8` (19200), on the edge.
5. **Match WinDNC flow control to XON/XOFF** (hardware/RTS-CTS off) — CNC has `P010=ON`; PC must match.
6. **Verify the cable pinout**: `2↔3, 3↔2, 5↔5`, and `4-6-8` shorted at *each* DB9 hood. Open the Harting hood to confirm.
7. **Swap USB-serial adapter for FTDI FT232** if the laptop currently uses a Prolific PL-2303 (clone chips are widespread and buggy).
