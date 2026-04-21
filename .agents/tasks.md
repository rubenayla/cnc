<!-- read in full — kept under 150 lines -->

# tasks.md

Shared kanban for Ruben + AI agents. Protocol: read before editing, claim before starting (move to In Progress with date + agent ID), atomic edits (one task at a time), mark Done only once verified.

Tag each task with **[ruben]** or **[ai]** to indicate who's best placed to do it. Either side can help, but the tag is the default owner.

## TODO

- [ruben] **Gather machine data**: exact Fagor 8055 submodel (8055i? 8055M? 8055T? TC?), firmware version shown on boot screen, and a clear photo of the label on the back of the operator panel. Needed to pick the right manual edition.
- [ruben] **Photograph the serial/Ethernet ports on the back of the operator panel**: labels visible (X1/X2/SERIE 1/etc.), plus what's currently plugged in. This decides the pinout conversation.
- [ruben] **Photograph the current WinDNC config dialogs**: Setup → Connection (protocol, port, baud, parity, stop bits), and the profile selected for this CNC. Also note WinDNC version (Help → About).
- [ruben] **Capture the CNC-side DNC state**: screenshot of the 8055 screen showing whether `DNC` / `DNC E` appears top-right during a transfer attempt, and the error number if any. Plus machine parameters `DNCTYPE` and `PROTOCOL` values if you can reach them (F7 → setup → machine parameters → serial lines).
- [ruben] **Confirm connection type**: is the laptop connected to the CNC via RS-232 serial cable (+ USB-serial adapter?) or Ethernet? If serial, take a photo of both ends of the cable and the adapter.
- [ruben] **Capture a failing WinDNC log**: after reproducing the red-text symptom, zip `C:\Fagor\WinDnc\` and drop it here under `logs/YYYY-MM-DD_red_text/windnc_log.zip`. Include a screenshot of the red text.
- [ruben] **Set up SSH into the Windows laptop** — follow `connection/ssh_setup.md` (AI will create this file once peer is ready). Requires peer to run PowerShell-as-admin commands to install OpenSSH Server. Give me the laptop's IP + username once sshd is running and I'll take it from there.
- [ai] **Write `connection/ssh_setup.md`**: step-by-step PowerShell instructions for the peer to enable OpenSSH Server on Windows, open firewall port 22, set sshd to autostart, make PowerShell the default shell, and share IP + username. Include the Mac-side `ssh-copy-id` + `~/.ssh/config` snippet.
- [ai] **Populate `docs/README.md`**: list the Fagor PDFs that should live in `docs/` (8055 Operating Manual, 8055 Installation Manual with serial/Ethernet params, WinDNC setup guide, 8055 error code list) with official Fagor download links. Don't commit the PDFs themselves until we know if we want them in git vs gitignored.
- [ai] **Write `connection/network_map.md` template**: placeholders for CNC IP, laptop IP, subnet, SSH host alias, WinDNC port/baud. Ruben fills in values as he gathers them.
- [ai] **Draft a `logs/README.md`** explaining the dated-subfolder convention and what evidence a good debugging log should contain (WinDNC log, screenshot of error, what was tried, CNC-side state, outcome).
- [ai] **Once SSH is up, do first remote inspection**: ssh into `cnc-laptop`, locate `C:\Fagor\WinDnc\`, read any `.log` files, run `Get-PnpDevice -Class Ports` to enumerate COM ports, and write findings to `logs/YYYY-MM-DD_first_remote_session/notes.md`.

## In Progress

_(empty — claim a task by moving it here with `[YYYY-MM-DD owner]` prefix)_

## Done

- [2026-04-21] **Scaffold repo**: created `~/repos/cnc/` with `AGENTS.md`, `README.md`, `.agents/tasks.md`, `docs/`, `logs/`. Layout documented in `AGENTS.md`.
- [2026-04-21] **Create public GitHub remote**: https://github.com/rubenayla/cnc pushed as `origin/main`.
