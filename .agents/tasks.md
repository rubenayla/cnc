<!-- read in full — kept under 150 lines -->

# tasks.md

Shared kanban for Ruben + AI agents. Protocol: read before editing, claim before starting (move to In Progress with date + agent ID), atomic edits (one task at a time), mark Done only once verified.

Tag each task with **[ruben]** or **[ai]** to indicate who's best placed to do it. Either side can help, but the tag is the default owner.

## TODO

- [ruben] **Photo-verify Fagor version**: Ruben confirmed on-site 2026-04-24 that it's 8055-M. For audit trail, photograph either (a) the `UTILIDADES → VERSION` screen (shows `FAGOR 80xx VERSION n.nn` + firmware rev) or (b) the label on the back of the operator panel (stamped model code). Once done, this closes out the 8050-vs-8055 trail in `docs/machine.md` and pins down the exact firmware for manual-edition matching.
- [ruben] **Read `P22`** (installation/machine parameters → general params): decides whether DNC rides on `X3` (RS-232) or `X4` (RS-422). Given Line 2 + Harting labelled "RS232", a mismatch here is the most suspicious single item on the parameter list. Document the value found and whether it matches the cabinet cable routing.
- [ruben] **Open the "RS232 DNC" Harting hood** on the cabinet and photograph the DB connector inside + how it's wired. Verify: `2↔3, 3↔2, 5↔5`, and `4-6-8` shorted at *each* hood. This is the canonical 8055 null-modem pinout from forum consensus.
- [ruben] **Drop baud to 9600 (both CNC and WinDNC) and retest.** Early 8055s can't run 19200 reliably — 9600 first, raise later only if everything else is stable.
  - **CNC side — from main menu:**
    1. `MAIN MENU` key
    2. `F7` (arrow / "more options") to reach second page of soft-keys
    3. `PARAMETROS MAQUINA` soft-key (may be abbreviated `PARAM. M.`)
    4. `LINEAS SERIE` soft-key
    5. `LINEA SERIE 2` soft-key — **verify title bar reads `PARAM. LINEA SERIE 2`, not SERIE 1**. DNC is on Line 2.
    6. Highlight `P000 BAUDRATE` (top row, usually already selected)
    7. `F2 MODIFICAR` (or `F1 EDITAR` if F2 does nothing — wording varies by firmware)
    8. `CL` to wipe the current `008`
    9. Type `7` on the numeric keypad
    10. `ENTER` — row should now read `P000 | 007 | BAUDRATE`
    11. `F6 SALVAR` — writes to non-volatile memory. **Do not skip**: with the backup battery dead, unsaved changes evaporate at power-cycle.
    12. `ESC` out, then press `RESET` (or full power-cycle) so the DNC task re-reads the params.
  - **PC side (WinDNC):** Setup → Connection → change this CNC's profile baud from 19200 to 9600. Leave everything else: 8 / N / 1, Fagor protocol, XON-XOFF flow control.
  - **Test:** ask WinDNC to read the CNC's directory (shortest round-trip). If listing loads → port layer is healthy at 9600. If screen still goes red → 9600 isn't the problem; next suspect is `P22`.
  - **Log result:** create `logs/2026-04-24_baud_9600/notes.md` (or whatever date) noting: prior value (008), new value (007), whether RESET was enough or power-cycle was needed, what WinDNC did after, any error numbers shown on the CNC.
- [ruben] **Reinstall WinDNC** on the Windows laptop before any on-site parameter work. Cheap first step; corrupted installs have been reported to produce persistent red-background even when everything else is correct.
- [ruben] **Identify the USB-serial adapter chipset** (Windows Device Manager → COM ports → properties → driver / hardware ID). If it's Prolific PL-2303, swap for an FTDI FT232-based cable — the Fagor-friendly "CNC-SW-25M" on eBay is pre-wired.
- [ruben] **Re-shoot INTZA nameplate**: current photo doesn't show TIPO or Nº clearly. Need a close-up, flash off, with raking light so the stamped model + serial are legible. Needed before we can contact INTZA for a manual.
- [ruben] **Email Holke for schematics** (`comercial@eholke.com`, in Spanish). They still exist at +34 943 310 744 — the nameplate phone number still works. Ask for: operator manual for F 2230 Nº 997, **Esquema Nº 011001**, parts list. Attach `resources/photos/nameplate.jpg`. Their maintenance arm is SEMAVE.
- [ruben] **Email INTZA** (`intza@intza.com`) once the TIPO is legible — ask for operation manual + drop-in replacement if EOL + spare sight-glass / refill-filter part numbers.
- [ruben] **Replace CPU-board backup battery** on the 8050 — clock is frozen at 01 Julio 2022, meaning the lithium is dead and parameters could be next. Identify battery type before visit.
- [ruben] **Photograph the serial/Ethernet ports on the back of the operator panel**: labels visible (X1/X2/SERIE 1/etc.), plus what's currently plugged in. This decides the pinout conversation.
- [ruben] **Photograph the current WinDNC config dialogs**: Setup → Connection (protocol, port, baud, parity, stop bits), and the profile selected for this CNC. Also note WinDNC version (Help → About).
- [ruben] **Capture the CNC-side DNC state**: screenshot of the 8055 screen showing whether `DNC` / `DNC E` appears top-right during a transfer attempt, and the error number if any. Plus machine parameters `DNCTYPE` and `PROTOCOL` values if you can reach them (F7 → setup → machine parameters → serial lines).
- [ruben] **Confirm connection type**: is the laptop connected to the CNC via RS-232 serial cable (+ USB-serial adapter?) or Ethernet? If serial, take a photo of both ends of the cable and the adapter.
- [ruben] **Capture a failing WinDNC log**: after reproducing the red-text symptom, zip `C:\Fagor\WinDnc\` and drop it here under `logs/YYYY-MM-DD_red_text/windnc_log.zip`. Include a screenshot of the red text.
- [ruben] **Set up SSH into the Windows laptop** — follow `connection/ssh_setup.md` (AI will create this file once peer is ready). Requires peer to run PowerShell-as-admin commands to install OpenSSH Server. Give me the laptop's IP + username once sshd is running and I'll take it from there.
- [ai] **Write `connection/ssh_setup.md`**: step-by-step PowerShell instructions for the peer to enable OpenSSH Server on Windows, open firewall port 22, set sshd to autostart, make PowerShell the default shell, and share IP + username. Include the Mac-side `ssh-copy-id` + `~/.ssh/config` snippet.
- [ai] **Write `connection/network_map.md` template**: placeholders for CNC IP, laptop IP, subnet, SSH host alias, WinDNC port/baud. Ruben fills in values as he gathers them.
- [ai] **Draft a `logs/README.md`** explaining the dated-subfolder convention and what evidence a good debugging log should contain (WinDNC log, screenshot of error, what was tried, CNC-side state, outcome).
- [ai] **Once SSH is up, do first remote inspection**: ssh into `cnc-laptop`, locate `C:\Fagor\WinDnc\`, read any `.log` files, run `Get-PnpDevice -Class Ports` to enumerate COM ports, and write findings to `logs/YYYY-MM-DD_first_remote_session/notes.md`.

## In Progress

_(empty — claim a task by moving it here with `[YYYY-MM-DD owner]` prefix)_

## Done

- [2026-04-21] **Scaffold repo**: created `~/repos/cnc/` with `AGENTS.md`, `README.md`, `.agents/tasks.md`, `docs/`, `logs/`. Layout documented in `AGENTS.md`.
- [2026-04-21] **Create public GitHub remote**: https://github.com/rubenayla/cnc pushed as `origin/main`.
- [2026-04-24] **Document machine from on-site photos**: added `docs/machine.md` (nameplate + control ID), `resources/photos/` with 7 resized JPEGs + index, `docs/README.md` with PDF policy and URL index.
- [2026-04-24] **Fetch Fagor 8055-M manuals**: downloaded 6 PDFs (errors, examples, ordering handbook committed; installation, operating, programming kept local-only via `.gitignore` due to size). All URLs catalogued in `docs/README.md`.
- [2026-04-24] **Reorganize into resources/**: moved photos and manuals out of `docs/` (which is now markdown-only) into `resources/photos/` and `resources/manuals/`, each with its own README.
- [2026-04-24] **Correct control ID to 8055-M**: photo-read of boot screen was ambiguous (0 vs 5); Ruben confirmed on-site it's 8055-M (milling variant). `machine.md`, `AGENTS.md`, `README.md`, `notes.md` updated; VERSION-screen photo still pending for final audit trail.
