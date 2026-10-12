---
purpose: The drawing standard extracted from the author's reference package — package structure, page types, conventions and measured style — de-identified; the package itself is private
read-when: Working on the sheet planner, renderer, DXF export or the author profile
status: draft (extracted Oct 12 2026)
---

# Reference package: extracted standard

The author supplied an as-built electrical package from a real project (60 sheets, 11 × 17,
AutoCAD LT, PDF and DXF). The documents stay in `examples/private/`, which is git-ignored (rule P8).
This note records only what defines the standard. Cabinet and device names below are generic.

The package covers a main control panel (**Cabinet A**) and three high-voltage cabinets (**B, C,
D**) on one EtherCAT network, with remote and field I/O, a TwinSAFE gate lock, CAN interfaces and
a VFD.

**Wire names are not taken from this package.** The author ruled that its wire labels are not the
standard; see CV2 and [ADR 0009](../docs/decisions/0009-numeric-wire-labels.md).

## 1 Package structure

```text
Cover: drawing list + general notes
Control system overview (EtherCAT topology, every module, cabinet boundaries, cable lengths)
Per cabinet, in fieldbus order (A, B, C, D):
    exterior layout · interior layout · harness overview (if any)
    power distribution 1..n
    I/O pages in slot order, with safety function pages right after the modules they use,
    typicals, and the field I/O chains that start in this cabinet
Device overviews (e.g. the VFD)
BOM: separate document, listed last in the drawing list
```

- The author's stated order (cover → exterior → interior → power → I/O per card) appears **once
  per cabinet**, with a system overview first and device overviews last. Adopted as CV3 (ADR 0010).
- No terminal plans, wire lists or cable schedules in the drawing package; wires carry gauge,
  colour and label on the schematic.

## 2 Page types

| Page type | What it shows | Maps to |
|---|---|---|
| Cover | Drawing list numbered by position (no drawing IDs; LCAD now generates them, CV4); general notes (ferrule or ring terminals; label both ends of every wire; label every device on or above it) | PS-5 |
| Control system overview | Network topology: controller, switch, couplers, every module with tag and part number, safety modules marked "F", cabinets as dashed boundaries, cable lengths, port-to-port labels `<dev>.<port>/<dev>.<port>` | PS-5 |
| Exterior layout | Front and side views, dimensions mm [in], cable entries, connectors labelled | exterior |
| Interior layout | Rails with device front views, tags above each device, rail and zone dimensions, type references | interior |
| Harness overview | Exploded isometric of a connector assembly with callouts | PS-7 |
| Power distribution | Feed through electronic-breaker channels to every device's power terminals; daisy-chains as jumpers; PE drops; HV DC bus with current transformers and shields | PS-4 |
| I/O module (cabinet) | Module as a box with part number and description; channel names; terminal numbers; signal descriptions on the left; field devices on the right; unused channels drawn; safety address DIP switches under the module; notes pointing to power and safety pages; cabinet boundary where a circuit leaves the cabinet; OEM contact diagram image where useful | PS-2 schematic |
| Safety function | Channels from two safety modules, the safety device with its isometric view, the cable with cores as colour + number | PS-3 |
| Typical | One drawing for eight identical circuits, placeholders `[W] [X] [Y] [Z]`, cross-reference table, wildcard title (`RIO120#`) | PS-6 |
| Field I/O | Isometric of the IP67 module and every device, cables named per port, default cable note; several small modules per page | PS-2 pictorial |
| Device overview | VFD terminals, links to other drawings, OEM installation steps and torque table | PS-1 |

## 3 Conventions observed (proposed for the author profile)

| Topic | Observed | Status |
|---|---|---|
| I/O module tags | `RIO` + 2-digit station + 2-digit slot, coupler = slot `00`; `FIO` + 2-digit segment + 2-digit position | CV1, adopted |
| CIO | Not used (the controller had no attached terminals) | consistent |
| Safety terminals | Marked "F" on the topology overview | to confirm |
| Device tags | Function abbreviation + number, no prefix sign (`PLC10`, `CR10`, `PT10`, `VFD1`); the tens digit seems to group related devices | to confirm |
| Wire annotation | Three stacked lines at the wire: gauge, colour, label | to confirm |
| Wire colours | BLU = +24 V DC and DC signals; WHT = 0 V; RED/BLK = main DC feed +/−; GRN = ground; ORG on HV-cabinet circuits | to confirm |
| Gauges | #20 signals and 24 V branches; #14 breaker channel feeds; #10 cabinet feed; #16 device PE; #3 cabinet ground | to confirm |
| Cable tags | `CBL-<device>-<port>`, `CBL-<function>`; cores `<cable>(<n>)` or colour + number | to confirm |
| Type references | `<THING>_TYP<n>` (terminal block, cable entry, connector types) | to confirm |
| Terminal references | `TB<group>-<function>` | to confirm |
| Revisions | Letters before release (A = initial release), numbers after (1 = as-built); dates `YYYY/MM/DD` | to confirm |
| Drawing titles | `CABINET <name>` / `<drawing type>` / `<subject>`; multi-sheet drawings `n OF m`; scale `N.T.S` | to confirm |

## 4 Measured style (from the DXF)

| Item | Value |
|---|---|
| Sheet | 17 × 11 in landscape; zone grid 8 columns (1–8) × 6 rows (A–F) |
| DXF structure | All sheets in model space at 1:1 mm; one paper-space layout per sheet (17 × 11 in) named after the drawing; interior layouts add a scaled viewport (about 1:4) |
| Font | ISOCPEUR |
| Text heights (printed) | 2.0 mm general; 1.5 mm small annotation; 5.0 mm large labels; title block 1.78–4.07 mm; cover title 13 / 20 mm |
| I/O channel pitch | 12.13 mm |
| Title block | Bottom right, about 127 × 43 mm: drawn / checked / approved / issued with initials and date; 5-row revision table; logo; title; drawing (3 lines); scale; sheet n of m; rev |
| Dimensions | mm with inches in brackets |
| CAD content | Vendor geometry exploded on vendor layers; no reusable symbol blocks |

## 5 Against RULES.md

| Rule | Finding |
|---|---|
| D1 | Consistent |
| DP1 | Refined: iso on safety, field I/O, device and harness pages; not on cabinet I/O or power pages |
| DP2–DP5 | Consistent; DP3 refined (field modules grouped), DP4 refined (inline placement) |
| DP6 | References named the drawing only; ruled: add drawing ID, sheet and zone (ADR 0010) |
| CV1 | Consistent |
| CV2 | Package labels rejected by the author; ADR 0009 defines the scheme |
| CV3 | Per-cabinet structure adopted (ADR 0010) |
| T1, T2, T4 | Answered (see `docs/drawing-philosophy.md`) |
