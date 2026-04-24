<!-- reference — read when relevant -->

# resources/photos/

On-site photos of the machine. Referenced from `docs/machine.md` and `.agents/notes.md`.

| File                        | Shows                                                                  |
|-----------------------------|------------------------------------------------------------------------|
| `nameplate.jpg`             | CE nameplate: Holke F 2230, Nº 997, 1998, 8000 kg, 400 V / 50 Hz       |
| `cnc_boot_screen.jpg`       | Fagor CNC boot screen, owner ONCE UTT Madrid, frozen clock             |
| `dnc_line2_parameters.jpg`  | `PARAM. LINEA SERIE 2` — the DNC serial-line params (P000–P010)        |
| `cabinet_main_switch.jpg`   | Main rotary disconnect on cabinet door + key bypass                    |
| `cabinet_side_aux.jpg`      | Side panel with "ANULA…" pushbutton and a Harting connector            |
| `intza_lubricator.jpg`      | INTZA centralized lubricator with sight gauge + pressure dial          |
| `rs232_dnc_connector.jpg`   | Harting HAN hood labelled "RS232 DNC" — where the WinDNC cable plugs in |

All images resized to ≤1800 px long side, JPEG q85. Originals (HEIC / full-res JPEG) are **not** kept in the repo.

## Adding a new photo

- Lowercase, underscore-separated, descriptive filename. `cabinet_rear_ports.jpg`, not `IMG_1234.jpg`.
- Resize before committing: `sips -Z 1800 -s formatOptions 85 -s format jpeg input.HEIC --out output.jpg`.
- Add a row to the table above and link it from `docs/machine.md` (or a dated `logs/YYYY-MM-DD_*/` folder if the photo is session-specific evidence rather than reference material).
