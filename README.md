# cnc

Troubleshooting and documentation for a **Fagor 8055-M SERCOS** CNC on a **Holke F 2230** vertical machining center (1998), owned by ONCE UTT Madrid. See `docs/machine.md` for the full identification trail.

> "-M" = milling variant (as opposed to -T for lathes). Confirmed on-site by Ruben 2026-04-24; photo of the VERSION screen / panel-rear label still pending for final verification.

## Current situation
- Control: Fagor 8055-M SERCOS (milling variant; operator panel + drive rack with APS-24 power supply, SERCOS fiber to servo/spindle drives)
- Host driving the CNC: a Windows laptop running WinDNC, WSL (Ubuntu) installed
- Symptom: when typing commands in WinDNC, text turns red — command transmitted but CNC doesn't ACK. CNC-side serial parameters confirmed correct (19200 / 8 / N / 1 / Fagor protocol / DNC auto-on / XON-XOFF). Current leading suspect: flow-control mismatch on the WinDNC side. See `.agents/history.md`.
- Goal: troubleshoot remotely from Ruben's Mac via SSH into the Windows laptop (OpenSSH Server on Windows, not WSL)

## Layout
- `AGENTS.md` — project instructions for agents (read first)
- `.agents/` — agent knowledge base
  - `.agents/tasks.md` — shared kanban (human + AI)
  - `.agents/history.md` — dated audit trail (what we found and decided)
  - `.agents/notes.md` — timeless reference (how things work)
- `docs/` — markdown notes only
  - `docs/machine.md` — machine identification (nameplate, control, aux systems)
  - `docs/README.md` — URL index + PDF policy
- `resources/` — binary assets
  - `resources/photos/` — on-site photos (indexed in its own README)
  - `resources/manuals/` — vendor PDFs (indexed in its own README)
- `logs/` — dated subfolders per debugging session (WinDNC logs, screenshots, notes)
- `connection/` — SSH notes, network map (created once we have the data)

## Not in this repo
- Passwords, keys, or the Windows laptop password — keep in `~/.ssh/config` or macOS Keychain
