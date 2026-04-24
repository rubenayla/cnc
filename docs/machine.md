<!-- reference — read when relevant -->

# Machine identification

Data transcribed from on-site photos taken 2026-04-24 (`resources/photos/`). Control is **Fagor 8055-M SERCOS** per Ruben's on-site read (2026-04-24). Earlier scaffolding said just "8055" and my photo read of `cnc_boot_screen.jpg` looked like "8050" — the "0" and "5" are ambiguous on that CRT. The panel-rear label or the `UTILIDADES → VERSION` screen is the authoritative confirmation; until one of those is photographed, treat "8055-M" as the working assumption.

## Machine (mill)

From the CE nameplate (`resources/photos/nameplate.jpg`):

| Field        | Value                                    |
|--------------|------------------------------------------|
| Manufacturer | Enrique Holke S.L., San Sebastián, Spain |
| Model        | F 2230                                   |
| Maquina Nº   | 997                                      |
| Esquema Nº   | 011001                                   |
| Año / Year   | 1998                                     |
| Peso         | 8000 kg                                  |
| Volts        | 400 V                                    |
| Hz           | 50                                       |
| Kw           | (illegible — confirm on site)            |

Manufacturer contact on plate: TLF 34-43-310744, FAX 34-43-310745, Guipúzcoa, Spain. (Holke has since changed hands; modern successor unknown — TBD.)

## Control

From the CNC boot screen (`resources/photos/cnc_boot_screen.jpg`):

- **Fagor 8055-M SERCOS** (Ruben's on-site read; photo-verification of the VERSION screen / panel-rear label still pending)
- Boot banner line (as read off the photo): `F-2213O-FAGOR 80?0 SERCOS` — the digit between `8` and `0` is ambiguous; see top-of-file note
- "M" = milling variant of the 8055 (mill-specific canned cycles, ISO programming). Other suffixes: T = turning / lathe, MC = milling conversational, TC = turning conversational.
- Machine registration / firmware line: `NUMERO DE MAQUINA: 41397, FABRICADO EN SEPT-1998`
- Owner banner: **ONCE — UTT — MADRID**
- CRT clock frozen at `Viernes 01 Julio 2022 02:22:41` → backup battery on CNC almost certainly dead (known 8050/8055 failure mode; clock resets at every power-on). Replace CR-style lithium on the CPU board.

## Auxiliary systems

- **Lubrication**: INTZA centralized lubricator (`resources/photos/intza_lubricator.jpg`), pressure gauge visible, model/part number not yet captured.
- **Main disconnect**: rotary "Interruptor Principal" on cabinet door (`resources/photos/cabinet_main_switch.jpg`), keyed bypass lock beside it for opening cabinet energised.
- **Auxiliary cabinet panel** (`resources/photos/cabinet_side_aux.jpg`): yellow pushbutton labelled "ANULA…" (cut off in photo — complete the label on next visit), and a Harting connector.
- **DNC port**: Harting HAN connector on cabinet labelled **"RS232 DNC"** (`resources/photos/rs232_dnc_connector.jpg`) — this is where the WinDNC laptop cable plugs in. The female DB connector is hidden behind the Harting hood; pinout needs capturing when the hood is opened.

## Open questions (add to tasks.md if acted on)

- Confirm **8050 vs 8055** from the label on the back of the operator panel.
- Exact submodel (M / T / MC / TC) and firmware version.
- Full Kw rating from nameplate.
- Complete text of the "ANULA…" pushbutton and what it anulls (door interlock bypass? axis interlock?).
- INTZA model number.
- Open the "RS232 DNC" Harting hood and document the DB pinout actually wired (8050 uses its own non-standard null-modem; don't trust generic pinouts).
