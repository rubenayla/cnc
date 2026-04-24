<!-- read in full — kept under 150 lines -->

# tasks.md

Shared kanban for Ruben + AI agents. Protocol: read before editing, claim before starting (move to In Progress with date + agent ID), atomic edits (one task at a time), mark Done only once verified.

Tag each task with **[ruben]** or **[ai]**. Either side can help; the tag is the default owner.

## Current status — read this before picking anything up

**The DNC "red text" bug has TWO likely open causes**, one of which is confirmed:

1. **File content (confirmed)** — `1001.pim` is a broken Fagor 8025→8055 converter output (see `.agents/history.md` 2026-04-25 "File `1001.pim` is the bug" entry). 527 KB after junk-stripping vs 108 KB free memory, block numbers above the N9999 ceiling. Fix: re-generate cleanly from CAM, or use DNC execution mode instead of COPY.
2. **Cable pinout (open)** — the peer has been using a generic Amazon USB-C to RS-232 adapter + an unknown DB-9 cable. The Fagor 8050/8055 CNC-side DB-9 is non-standard (pin 7 = GND, not pin 5). Unless the cabinet's Harting hood has a Fagor adaptation pigtail inside, PC ground is landing on CNC CTS and the transfer is only working through chassis-ground coupling — intermittent by nature. See `.agents/history.md` 2026-04-25 "Second open issue" entry and `docs/rs232_pinout.md` for the correct pinout.

Port assignment, machine parameters (baud, bits, parity, flow control, protocol), and DNC-on-power-up are all verified correct. A **Holke/Fagor technician is coming on-site** to teach Ruben; most CNC-side tasks below are framed around that visit.

## Before the technician visit — prep (Ruben)

- **Minimal test program ready on the laptop.** Create `test.pim` on the Windows laptop containing exactly:
  ```
  %0999,M,
  (MSG "DNC OK")
  M30
  ```
  (three lines, CRLF line endings, saved as `test.pim` in WinDNC's working directory). This is the first thing to transfer during the visit — if it copies and P999 becomes a valid 3-block program, the whole DNC stack is healthy and the technician can focus on the real problem (workflow, not wiring).
- **Repo up to date on the laptop.** Clone or pull `github.com/rubenayla/cnc` on the Windows laptop so the technician can read `docs/rs232_pinout.md`, the manuals under `resources/manuals/`, and `.agents/history.md` while on-site. Run `git pull` the morning of the visit.
- **WinDNC version captured.** Help → About → screenshot → save to `logs/YYYY-MM-DD_tech_visit/windnc_version.png`.
- **WinDNC log folder zipped.** `C:\Fagor\WinDnc\` — make a zip before the visit in case the technician clears it. Save to `logs/YYYY-MM-DD_tech_visit/windnc_before.zip`.
- **USB-serial adapter details captured.** Device Manager → COM ports → properties → hardware ID (FTDI / Prolific / CH340 / CP2102). Photograph the physical adapter, both ends. **Note the model / Amazon listing name if possible** — the generic ones expose standard DB-9 PC pinout, which does NOT match the CNC's non-standard pinout unless a Fagor pigtail sits between them.
- **Photograph the current cable path end-to-end.** From the laptop USB-C port → USB-to-RS-232 adapter → DB-9 cable → Harting hood → cabinet. One photo of each link. This is the evidence the technician needs to diagnose the wiring suspect without having to re-trace it themselves.
- **Battery type identified.** Pre-buy if possible: expected is a 3.6 V Saft LS14500 (AA-sized) lithium on the 8055 CPU board. Confirm with the technician before swapping.
- **Questions list ready** (see below).

## Questions to ask the technician on-site

1. **`1001.pim` workflow.** Is the 8025→8055 converter supposed to produce runnable output, or is it only for reference? What's the peer's intended CAM-to-CNC chain? (ideally bypass the 8025 path entirely)
2. **DNC execution mode** — can the 8055-M execute large programs streamed block-by-block over Line 2 without storing them in EEPROM? What's the keystroke to enter that mode? (operator manual chapter reference appreciated)
3. **EEPROM size and upgrades.** The directory screen shows ~108 KB free. Is there an option for extended memory, or is 108 KB the hard ceiling for this hardware revision?
4. **Block-number ceiling.** Is N9999 the hard max on this firmware, or can it be extended in machine parameters?
5. **Backup battery replacement.** Exact part number. Procedure — does the CNC need to be powered on during swap to retain parameters, or is a full parameter backup + restore required?
6. **Parameter backup procedure.** What's the canonical way on this 8055-M to dump **all** machine parameters (general + axes + spindle + SERCOS + serial + PLC) to a single file over DNC? Best format?
7. **Firmware version.** What firmware revision is installed? Is an upgrade available/recommended for this hardware?
8. **P22 value** — ask the technician to read and note it, to close the audit trail. It should route DNC to Line 2.
9. **Harting hood pinout** — have them open the cabinet Harting labelled "RS232 DNC" and photograph the DB connector inside. **This is now an active suspect, not a formality**: the peer has been using a generic Amazon USB-C to RS-232 adapter (standard DB-9 pinout), but the CNC is non-standard — if there's no Fagor adaptation pigtail inside the hood, PC ground lands on CNC CTS and CNC ground is unterminated. That would explain the intermittent/partial transfers even before the `1001.pim` content issue kicks in. Confirm whether it matches the legacy-Fagor pinout in `docs/rs232_pinout.md` or the standard-DB9 pinout.
10. **Long-term support.** Is Fagor still selling spares for this control? What's the realistic upgrade path if the CPU board fails (retrofit to 8055i? 8065?)?
11. **Lubrication — INTZA.** Have the technician glance at the INTZA unit and tell us the model (if readable) and the recommended oil grade + top-up interval while he's here.
12. **Safety door interlock.** The UTILIDADES screen shows the warning `6 SEGURIDAD PUERTAS ANULADA` (door-safety interlock bypassed). Is that an operator-choice override, a stuck sensor, or a temporary workaround? Should we restore it?

## On-site with the technician — order of operations

- **First action:** transfer `test.pim` with WinDNC to verify the DNC stack. If it works → move on. If it fails → technician diagnoses from scratch.
- **Second action:** full parameter backup (via DNC) before the technician changes anything. Save to `logs/YYYY-MM-DD_tech_visit/params_before.txt`.
- **Third action:** battery replacement with the technician's supervision.
- **Fourth action:** walk through the questions list with the technician. **Write down verbatim answers** (don't paraphrase in the moment — you'll lose nuance).
- **Throughout:** photograph every new screen the technician opens. Every error number, every sub-menu, every parameter page.

## After the technician leaves — capture (Ruben + AI)

- **[ruben] Scan the written notes**; save as `resources/photos/tech_notes_YYYY-MM-DD.jpg`.
- **[ai] Transcribe the tech's answers** into `.agents/notes.md` under each relevant section. Also write a new `history.md` entry dated with the visit date summarising what was learned and what changed (params edited, battery swapped, etc.).
- **[ruben+ai] Resolve or close the pending tasks** that the visit answered. If P22 gets read, close the P22 task. If the tech confirms the Harting pinout, close the "open Harting hood" task.
- **[ai] Update `docs/rs232_pinout.md`** if the tech's pinout reading disagrees with either of the two documented variants.

## Secondary tasks (do whenever)

- [ruben] **Photo-verify Fagor version** — `UTILIDADES → VERSION` screen or the rear-panel stamped model code. Closes the 8050-vs-8055 audit trail in `docs/machine.md`.
- [ruben] **Re-shoot INTZA nameplate** — close-up, flash off, raking light, TIPO + Nº legible. Unblocks the INTZA manual request.
- [ruben] **Email Holke for schematics** (`comercial@eholke.com`, Spanish, attach `resources/photos/nameplate.jpg`). Request: operator manual for F 2230 Nº 997, **Esquema Nº 011001**, parts list. Their maintenance arm is SEMAVE.
- [ruben] **Email INTZA** (`intza@intza.com`) once the TIPO photo is legible.
- [ruben] **Photograph the serial/Ethernet ports on the back of the operator panel** (X1/X2/SERIE 1 labels), plus what's plugged in where.
- [ruben] **Set up SSH into the Windows laptop** — install OpenSSH Server on Windows (not WSL). Give me IP + username once sshd is running.
- [ai] **Write `connection/ssh_setup.md`** — PowerShell commands for the peer to enable sshd, open firewall port 22, set it to autostart. Include Mac-side `~/.ssh/config` snippet.
- [ai] **Write `connection/network_map.md` template** — placeholders for CNC IP, laptop IP, SSH host alias, WinDNC port/baud.
- [ai] **Draft `logs/README.md`** — the dated-subfolder convention and what a good session log contains.
- [ai] **Once SSH is up, first remote inspection** — `C:\Fagor\WinDnc\` log files, `Get-PnpDevice -Class Ports` for COM enumeration. Write findings to `logs/YYYY-MM-DD_first_remote_session/notes.md`.

## In Progress

_(empty — claim a task by moving it here with `[YYYY-MM-DD owner]` prefix)_

## Done

- [2026-04-21] **Scaffold repo**: created `~/repos/cnc/` with `AGENTS.md`, `README.md`, `.agents/tasks.md`, `docs/`, `logs/`. Layout documented in `AGENTS.md`.
- [2026-04-21] **Create public GitHub remote**: https://github.com/rubenayla/cnc pushed as `origin/main`.
- [2026-04-24] **Document machine from on-site photos**: added `docs/machine.md` (nameplate + control ID), `resources/photos/` with 7 resized JPEGs + index, `docs/README.md` with PDF policy and URL index.
- [2026-04-24] **Fetch Fagor 8055-M manuals**: downloaded 8 PDFs (all committed after Ruben's call).
- [2026-04-24] **Reorganize into resources/**: moved photos and manuals out of `docs/` (markdown-only) into `resources/photos/` and `resources/manuals/`.
- [2026-04-24] **Correct control ID to 8055-M**: Ruben confirmed on-site; `machine.md`, `AGENTS.md`, `README.md`, `notes.md` updated.
- [2026-04-25] **Audit Downloads folder**: 6 additional on-site photos of the DNC attempt moved into `resources/photos/` with descriptive names; originals + already-processed files deleted from `~/Downloads`.
- [2026-04-25] **Identify root cause of "red text"**: `1001.pim` is a broken Fagor 8025→8055 converter output (too big, invalid block numbers). Wire is fine. Full writeup in `.agents/history.md`.
- [2026-04-25] **Document RS-232 pinout**: `docs/rs232_pinout.md` with CNC DB-9, PC DB-9, PC DB-25 side diagrams + the two pinout variants seen in the wild.
