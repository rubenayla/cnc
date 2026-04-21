# cnc

Troubleshooting and documentation for a Fagor 8055 CNC belonging to a friend.

## Current situation
- Control: Fagor 8055 (operator panel + drive rack with APS-24 power supply, SERCOS fiber to servo/spindle drives)
- Host machine driving the CNC: a Windows laptop running WinDNC, with WSL (Ubuntu) already installed
- Symptom under investigation: when typing commands in WinDNC, text turns red — command sent but the CNC does not ACK (half-working comms, likely DNC not enabled, wrong protocol/baud, bad cable pinout, or flaky USB-serial adapter)
- Goal: troubleshoot remotely from Ruben's Mac via SSH into the Windows laptop (OpenSSH Server on Windows, not WSL), so Claude Code can drive the remote shell

## Layout
- `AGENTS.md` — project instructions for agents (read first)
- `.agents/tasks.md` — shared task board (human + AI)
- `docs/` — Fagor manuals, WinDNC guide, error code list (not yet populated)
- `logs/` — dated subfolders per debugging session (WinDNC logs, screenshots, notes)
- `connection/` — SSH notes, network map (created once we have the data)

## Not in this repo
- Passwords, keys, or the Windows laptop password — keep in `~/.ssh/config` or macOS Keychain
