<!-- reference — read when relevant -->

# resources/manuals/

Vendor PDFs, all committed. For URL index + alternative editions, see `docs/README.md`.

| File                                    | Size   | Covers                                                                                           |
|-----------------------------------------|--------|--------------------------------------------------------------------------------------------------|
| `fagor_8050_rs232_setup.pdf`            | 220 KB | Fagor app note: serial-line params, null-modem pinout, WinDNC handshake (applies to 8050 + 8055) |
| `fagor_8055_installation.pdf`           | 9.7 MB | **Installation / OEM manual** — machine parameters (`DNCTYPE`, `PROTOCOL`), serial pinouts, SERCOS integration. **Highest priority for the current WinDNC bug.** |
| `fagor_8055m_operating.pdf`             | 3.8 MB | Operating manual, **English** — contains the WinDNC chapter.                                     |
| `fagor_8055m_operating_es.pdf`          | 3.8 MB | Operating manual, **Spanish** — same content as EN. Matches the CNC's UI language.               |
| `fagor_8055m_programming.pdf`           | 6.2 MB | Programming manual — ISO G-code, canned cycles (G81–G89), pocket milling, subroutines.          |
| `fagor_8055m_errors.pdf`                | 0.6 MB | Error-solving manual. Numeric error codes + PLC/SERCOS diagnostic families.                      |
| `fagor_8055m_examples.pdf`              | 1.5 MB | Worked programming examples (milling).                                                           |
| `fagor_8055i_ordering_handbook.pdf`     | 0.3 MB | 8055i ordering handbook — part numbers + options for spares.                                     |

Total: ~26 MB. Per-PDF source URLs are in `docs/README.md` in case any need to be re-fetched or cross-checked against a later edition.

## Naming convention

Lowercase, underscores, no spaces. Prefix with vendor + product.

- `fagor_8055_*.pdf` — generic 8055 (covers both M and T)
- `fagor_8055m_*.pdf` — milling-specific
- `fagor_8055t_*.pdf` — turning-specific (not expected for this machine)
- `holke_f2230_*.pdf` — Holke manuals once we get them
- `intza_*.pdf` — INTZA once the TIPO is legible

## Not here yet

Per `.agents/tasks.md` — still waiting on:

- Holke F 2230 operator manual + **Esquema Nº 011001** (email `comercial@eholke.com`)
- INTZA lubricator manual (blocked on re-shooting the nameplate so we can identify the TIPO)
