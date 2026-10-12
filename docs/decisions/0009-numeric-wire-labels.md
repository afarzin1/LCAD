---
purpose: ADR — the author profile's wire-label scheme: six digits, one label per potential, I/O labels built from the module number and terminal
read-when: Touching wire labels, numbering, the numbering lock, or moving devices between cabinets
status: stable
---

# ADR 0009 — Numeric wire labels: area, group, sequence

**Status:** Accepted 2026-10-12. Resolves the CV2 conflict raised against the reference package.

## Context

The author stated that wire labels are all numeric (CV2). The author's reference package used a
mixed scheme (`1001-7` on I/O wires, `ECB101-P1`, `CAB-N1`, `RIO1000-24V` on power wires), which
contradicted CV2. The author ruled: the reference package is not the model for wire names; labels
must be **consistent, numeric, short and easy to follow**. The author likes I/O channels following
their module (module 1001 → `1001-1` … `1001-8`) but **scalability wins** where the two disagree.

Requirements drawn from that:

1. Digits only, no separators, one fixed length.
2. Every wire in the project follows the same rule, including power and interlock wires.
3. A technician can tell from the label where to look.
4. Scales to large machines without running out of numbers or changing format.
5. Allocated deterministically and stable once released [M10].

## Decision

Every label is **six digits: `AA GG SS`**, one label per **potential** (every conductor on an
electrically common node carries the same number).

| Part | Digits | Meaning |
|---|---|---|
| `AA` area | 2 | For I/O: the module's station number (the `10` in RIO1001). For other circuits: the cabinet's area number |
| `GG` group | 2 | `01`–`89`: the I/O module's slot (the `01` in RIO1001). `90`–`99`: non-I/O circuit groups in that area (power distribution, interlocks, control circuits) |
| `SS` sequence | 2 | For I/O: the module terminal (pin index from the part record). For circuit groups: allocated in drawing order |

**Examples**

| Wire | Label | Reads as |
|---|---|---|
| RIO1001 terminal 7 to a relay coil | `100107` | RIO1001, terminal 07 |
| RIO1101 terminal 12 | `110112` | RIO1101, terminal 12 |
| First 24 V branch in cabinet area 10 | `109001` | area 10, power group 90, wire 01 |
| Field module FIO2001 port pin wired by hand | `200103` | FIO2001, pin index 03 |

So the first four digits of an I/O label are always the module number, which keeps the pattern the
author likes, without a dash.

**Which family a potential uses** (deterministic): a potential connected to exactly one I/O module
terminal and no supply takes that terminal's I/O label. Every other potential (supplies, returns,
commons, interlock chains between devices) takes a circuit-group label in the area of its source.

**Capacity:** 99 areas × 89 modules × 99 terminals, plus 10 circuit groups × 99 wires per area. A
large cabinet with more than 990 non-I/O potentials is given a second area number.

**Constraints:** station numbers are unique across RIO and FIO, and cabinet area numbers do not
collide with stations used elsewhere (checked by an invariant). Labels are allocated once and
locked; rebuilding never renumbers [M10].

## Options considered

- **Module-dash-channel (`1001-1`)**: readable, but has a separator and no rule for power wires.
- **Pure sequential (`1`, `2`, `3` …)**: short and scalable, but the label says nothing about where
  the wire is.
- **Drawing/line based (`0712` = drawing 7, line 12)**: common, but drawing numbers move when sheets
  are inserted, so the pattern decays even if labels are locked.
- **Area/group/sequence (chosen)**: one format for every wire; I/O labels embed the module number;
  scales by adding areas.

## Consequences

- Labels encode location (area). Moving a device to a cabinet with a different area number makes
  its circuit-group labels inconsistent with the scheme; that is finding F6 in `integrity.md`
  (renumber, or keep the released labels with a recorded exception).
- Cable tags, terminal tags and other identifiers are not affected by this decision.
- The reference package remains evidence for drawing style only, not for wire names.
