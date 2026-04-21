<!-- read in full — kept under 150 lines -->

# AGENTS.md — cnc

Fagor 8055 CNC troubleshooting repo. Owner: Ruben. Machine owner: a friend (peer operates the Windows laptop on-site).

## What this repo is for
1. Collecting Fagor 8055 / WinDNC documentation in one place
2. Recording SSH + network details for the Windows laptop that drives the CNC
3. Logging debugging sessions (WinDNC logs, screenshots, what was tried, what the CNC did)
4. Becoming a durable reference if the kart team (or anyone else) inherits this machine later

## How to work here
- Read `.agents/tasks.md` before starting. It's the shared board between Ruben and AI agents. Claim a task by moving it to In Progress with date + agent ID; don't grab something already claimed.
- Each debugging session gets its own dated folder under `logs/` (e.g. `logs/2026-04-21_red_text/`) with the raw evidence and a `notes.md` summary. Keep evidence together so a single folder read reconstructs the session.
- Docs go in `docs/` as PDFs. Don't commit huge PDFs to git history — if `docs/` grows past a few MB, gitignore it and keep a `docs/README.md` listing what should live there and where to get it.
- Never commit secrets. Windows laptop password, API keys, anything sensitive → macOS Keychain or `~/.ssh/config`, not this repo. If a log file contains credentials, redact before committing.

## Conventions
- Dates in filenames and log entries: `YYYY-MM-DD`.
- SSH host alias for the Windows laptop (once set up): `cnc-laptop` in `~/.ssh/config`. CNC itself has no shell (embedded; FTP only, default IP seen on display: 172.16.1.1).
- Don't invent facts about the machine. If a parameter, IP, or pinout is unknown, mark it `TBD` in notes and add a task to find out.

## Known facts (as of 2026-04-21)
- Control: Fagor 8055, operator panel with DB9/DB25 RS-232 (likely X2 / SERIE 1 on the back of the panel, not the drive rack)
- Drive rack: APS-24 power supply + servo + spindle drives, SERCOS fiber ring between CNC and drives
- APS-24 has its own small keypad/display (PS-25B4 style) that plugs into the PS front, unrelated to PC comms
- CNC exposes FTP over Ethernet (default on 172.16.1.1 in the earlier screenshot); no SSH on the CNC itself
- Windows laptop has WSL Ubuntu installed but SSH will target Windows directly, not WSL (WSL can't see COM ports without usbipd-win plumbing)

## Current symptom
WinDNC: when peer types commands, characters turn red — command transmitted but not ACKed by CNC. Comms half-working. Suspects in order: DNC not enabled on 8055, wrong PROTOCOL machine parameter, baud mismatch, non-standard 8055 null-modem pinout (2↔3, 3↔2, 5↔5, jumper 4-6-8 each end), USB-serial adapter (Prolific chips especially).
