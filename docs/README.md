<!-- reference — read when relevant -->

# docs/

Markdown notes only. Binary assets (photos, vendor PDFs) live under `resources/` instead — see `resources/README.md`.

Contents:

1. **`machine.md`** — identification of the physical machine. Start here.
2. **This file** — URL index and PDF policy.

## Where binary assets live

- **Photos**: `resources/photos/` (indexed in `resources/photos/README.md`)
- **Manuals**: `resources/manuals/` (indexed in `resources/manuals/README.md`)

Currently the only committed PDF is `resources/manuals/fagor_8050_rs232_setup.pdf` (220 KB Fagor app note on DNC serial parameters — the single most useful doc for the current red-text bug). Source: http://www.devgraphik.com/8050_RS-232.pdf

## PDF policy

All vendor manuals for this project are committed into `resources/manuals/`. Total is ~26 MB today, well under any GitHub limit and worth the tradeoff so a fresh clone gives anyone everything needed to work on this machine offline. Keep source URLs in the table below for edition checks and re-fetching.

If the pool ever balloons past ~100 MB, revisit: either move the biggest PDFs out and gitignore them, or migrate the assets to git-lfs.

## Fagor 8055-M SERCOS — manual set

The **8055 (rack)** and **8055i (integrated single-board)** share the same manual set; Fagor titles them "CNC 8055 / CNC 8055i". The `·M·` suffix is the milling software kernel (this machine), as opposed to `·T·` (lathe), `·MC·` / `·TC·` (conversational variants). Milling-specific content is in the Operating / Programming / Error / Examples manuals; the Installation manual is generic to M and T.

Status: the 8055 family is **previous-generation but still serviced**. Fagor's current line is 8058/8060/8065/8070 ("CNCelite"). WinDNC is still distributed and supported for 8055-family DNC; new machines use DNC over Ethernet via "CNC Explorer" instead.

Mirrors below are all login-free. Fagor's own downloads page (https://www.fagorautomation.com/en/downloads/) requires registration but hosts the authoritative latest editions.

### Committed to the repo (all in `resources/manuals/`)

| File                                    | Size   | Covers | Source URL (for re-fetching / edition checks) |
|-----------------------------------------|--------|--------|----------------------------------------------|
| `fagor_8050_rs232_setup.pdf`            | 220 KB | Fagor app note: serial-line params, null-modem pinout, WinDNC handshake. Applies to 8050 and 8055. | http://www.devgraphik.com/8050_RS-232.pdf |
| `fagor_8055_installation.pdf`           | 9.7 MB | **Installation / OEM manual** — machine parameters incl. `DNCTYPE`, `PROTOCOL`, serial-line params, null-modem pinout, SERCOS integration. **Highest priority for the WinDNC red-text bug.** | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Installation-Manual-English.pdf |
| `fagor_8055m_operating.pdf`             | 3.8 MB | Operating manual, **EN** — operator screens, MDI, jog/handwheel, **WinDNC chapter**. Ref. 1402, Soft V01.6x. | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Operating-Manual-English.pdf |
| `fagor_8055m_operating_es.pdf`          | 3.8 MB | Operating manual, **ES** — same content as EN. Matches the CNC's UI language. | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Operating-Manual-Spanish.pdf |
| `fagor_8055m_programming.pdf`           | 6.2 MB | Programming manual — ISO G-code, canned cycles (G81–G89), pocket milling, subroutines. | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Programming-Manual-English.pdf |
| `fagor_8055m_errors.pdf`                | 0.6 MB | Error-solving manual. Numeric error codes + PLC/SERCOS families. Ref. 1310. | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Error-Solution-English.pdf |
| `fagor_8055m_examples.pdf`              | 1.5 MB | Worked programming examples, milling. Ref. 1010. | https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Examples-Manual-English.pdf |
| `fagor_8055i_ordering_handbook.pdf`     | 0.3 MB | 8055i ordering handbook — part numbers, options. Useful for spares. Ref. 1111. | https://www.endo.com.tr/extras/dosya-merkezi/OTOMASYON/FAGOR/8055m/man_8055i_ord_hand.pdf |

### Other editions / languages / mirrors

- **Operating manual, older edition (Ref. 9909):** https://static1.squarespace.com/static/5f6720d257cb4d1732f2b451/t/5fd39a54dc4d121fc3c5b3b1/1607703140753/CNC-8055-M-Operator-Manual.pdf
- **Operating manual, Spanish:** https://dmscncrouters.com/wp-content/uploads/2016/05/Fagor-CNC-8055-Operating-Manual-Spanish.pdf
- **Installation manual, older edition (Ref. 0001):** http://isp.ljm.free.fr/manuels/fagor/ang/8055oem.pdf
- **Programming manual, newer edition (Ref. 1711, Soft V02.2x, 482 pp):** https://www.manualslib.com/manual/1384310/Fagor-8055-M.html (login to download)
- **Errors manual, Endo mirror (Ref. 1010):** https://www.endo.com.tr/extras/dosya-merkezi/OTOMASYON/FAGOR/8055m/man_8055m_err.pdf
- **Endo mirror root (login-free directory):** https://www.endo.com.tr/extras/dosya-merkezi/OTOMASYON/FAGOR/8055m/ — has `man_8055m_prg.pdf`, `man_8055t_prg.pdf`, `man_8055mco_user.pdf`, `man_8055i_ord_hand.pdf`
- **Fagor official landing (login required):**
  - https://www.fagorautomation.com/en/descargas/cnc-8055-installation-manual
  - https://www.fagorautomation.com/en/descargas/cnc-8055-m-programming-manual
- **MC (conversational) self-teaching manual:** https://www.masteel.ca/uploads/1/1/2/8/112811735/fagor_8055mc_self-teaching_manual.pdf — only relevant if this machine turns out to be `·MC·`, not `·M·`

### WinDNC

- **Installer (official, no login, ~2 MB ZIP):** https://www.fagorautomation.com/en/descargas/setup-windnc
- No standalone WinDNC manual — the WinDNC chapter lives inside `fagor_8055m_operating.pdf` (search: "WinDNC" or "DNC").
- WinDNC ≥ 4.1 required for HD-table transfers; supports up to 10 simultaneous connections.

### 8050 vs 8055 note

Until the VERSION screen or operator-panel rear label is photographed, "8055-M" is Ruben's on-site read. If it turns out to be 8050, the docs are organized differently (two big books: OEM + USER). 8050 mirrors are preserved here for reference:
- 8050 OEM (install + SERCOS + params + errors) EN: https://manualzz.com/doc/38321708/fagor-cnc-8050-oem-manual
- 8050-M USER (operating + programming) EN: https://manualzz.com/doc/23189917/fagor-cnc-8050-m-manual

## Enrique Holke F 2230 (machine)

**Holke is still in business** — same phone as the 1998 nameplate, email active. They are almost certainly the only source for **Esquema Nº 011001** (it does not appear online).

- Site: https://eholke.com/
- Sales email: **comercial@eholke.com**
- Phone: **+34 943 310 744** (same as on the nameplate)
- Maintenance/retrofit arm: **SEMAVE** (division of Enrique Holke S.L.) — https://mantenimiento-y-reparacion-de-maquinas-herramienta-semave.com/
- Address shift: originally San Sebastián, now Arroa-Zestoa (Gipuzkoa)

Note: F 2230 is a **vertical machining center**, not a knee-mill. Siblings in the same VMC line: F2233, F2242. Published specs for F 2230: Fagor 8050 CNC, 1200×500 table, X/Y/Z ≈ 1000/500/620 mm, ISO 40 spindle, ~17 kW, ~6000 kg (nameplate's 8000 kg likely includes pallet/coolant tank).

Tony Griffiths' lathes.co.uk archive sells manuals for the older Holke knee-mill line (F6-V through F15-CNC) — **wrong generation, not useful for F 2230**. Don't buy HM684.

### Request to send Holke

Email in Spanish, attach `resources/photos/nameplate.jpg`. Ask for:

- Operator manual for F 2230, Nº 997
- Electrical schematic Esquema Nº 011001
- Parts list / lubrication schedule
- Current contact for spares (even if they're EOL)

## INTZA lubricator

- Site: https://www.intza.com/ (still active)
- Email: **intza@intza.com**
- Phone: **+34 943 852 600**
- Address: Polígono Ugarte 15, 20720 Azkoitia (Gipuzkoa)

The existing photo's nameplate (**TIPO** and **Nº**) is not legible. **Re-shoot the plate** (close-up, flash off, raking light) before contacting INTZA — otherwise they can't identify the exact unit. The lubricator is a single-line oil system, ~30 bar working pressure, typical of 1990s Spanish machine tools; current catalog equivalent is the GE0x/GE1x/GE2x electric-pump family.

## Conventions

- File names: lowercase, underscores, no spaces. e.g. `fagor_8050_oem.pdf`, not `Fagor 8050 OEM.pdf`.
- When adding a PDF, add a one-liner in the relevant section above with the source URL (or "from vendor email YYYY-MM-DD" if received privately).
