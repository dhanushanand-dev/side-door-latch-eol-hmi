# System Overview

> **Note:** The source code for this project is kept in a private repository. File names below refer to that repository. Access is [available on request](https://github.com/dhanushanand-dev).

## Where the HMI fits

The end-of-line (EOL) station tests every side door latch before it leaves the line. The **PLC** runs the test sequence:

| Test | Purpose |
|---|---|
| Functionality | The latch operates through its full mechanical cycle |
| Engagement | The latch engages the striker correctly |
| Release | The latch releases when actuated |
| Auto-canceling | The auto-cancel function resets the latch as designed |

When a latch passes, the operator uses the **HMI** to print its traceability label. The label carries a unique serial number and a QR code, so the part can be traced back to its date, shift, variant and side.

## Operator flow

1. **Log in.** The available screens depend on the role (see the permissions below).
2. **Label printing.** Select the part variant. Part number, part name and model come from `system_config.json`.
3. Choose the **side** (LH / RH) and the **shift**. Only the shifts that match the current time can be selected. Then set the **quantity**.
4. Check the **label preview** and the **next serial number**.
5. Press **Print Labels**. The HMI generates the serials, prints them, advances the counter and writes each label to the print log.

## Serial number scheme

```
00009 05102026 B
└─┬─┘ └──┬───┘ └ shift letter (A, B, C, G)
  │      └ print date (DDMMYYYY)
  └ 5-digit counter, kept separately for each variant + side + shift
```

The counters are stored in `shift_serial_tracking.json`, so numbering carries on after a restart. The manual print panel can start from a chosen counter value, for example for a re-print.

## Label (ZPL)

The HMI writes the ZPL itself and sends it to the printer as a RAW job through the Windows spooler (`win32print`).

- Label size: 280 × 120 dots (`^PW280`, `^LL120`)
- **QR code** (`^BQN`) holding *part number + serial*
- A large vertical **LH / RH** side indicator
- Up to five text lines: station header, part number, part name, optional SAP number, and serial

## Shifts

| Shift | Time |
|---|---|
| A | 06:00 – 14:00 |
| B | 14:00 – 22:00 |
| C | 22:00 – 06:00 |
| G | 08:30 – 17:00 (general) |

## Roles and permissions

These are the defaults in `permissions.json`. An Admin can change them on the Maintenance screen.

| Role | Screens |
|---|---|
| Admin | All |
| Supervisor | Reports, Manual Control, Barcode Printing |
| Operator | Barcode Printing |
| Maintenance | Manual Control, System Settings |

## Data files

| File | Content |
|---|---|
| `system_config.json` | Variants and their part details, label types, sides, shifts, defaults |
| `permissions.json` | Role → allowed screens |
| `users.json` | Accounts and roles (create it from `users.example.json`; git-ignored) |
| `shift_serial_tracking.json` | Serial counters per variant/side/shift (created at runtime) |
| `print_logs.csv` | One row per printed label: date, time, user, variant, part no, side, label type, shift, mode, serial, quantity (created at runtime) |

## Reports

The **Print Report Viewer** filters `print_logs.csv` by date range and shift. It exports the result to:

- **Excel** (`.xlsx`, formatted header, via openpyxl)
- **PDF** (landscape A4 table, via reportlab)
