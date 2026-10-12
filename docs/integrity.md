---
purpose: How LCAD keeps every object relation consistent when the model changes — derived data, typed operations, global invariants, findings for the AI, and the drawn/placed/procured BOM cross-reference
read-when: Implementing or requesting any model change (move, rename, replace, delete, connect); designing validators; anything touching BOM completeness
status: draft
---

# Change integrity and cross-referencing

## 1 The problem

Request: *"Move -K12 to cabinet 2."* In a drawing-based tool, someone now has to remember every
place -K12 appears: its schematic sheets and cross-references, the panel layout of both cabinets,
the BOM location column, the wires that used to be internal and now cross between cabinets, the
terminals and interconnect cable those wires now need, cable entries, the power feed, wire numbers
that encode the cabinet, and the approvals that are now stale.

Two bad ways to handle this:

- **An LLM works through a checklist.** Expensive (it has to read the whole project to find
  occurrences) and unreliable (it will eventually miss one, and nothing notices).
- **Hand-written propagation code per operation.** Every new operation needs its own list of
  side effects, and the lists drift.

LCAD uses a deterministic approach with a narrow role for the AI.

## 2 Approach: three layers, AI last

| Layer | What it does | Who |
|---|---|---|
| **1. Derive, don't store** [M3] | Most "occurrences" are not stored at all, so there is nothing to update | Model design |
| **2. Typed operations** [I1, I2, I4] | Apply the change and every consequence that has one correct answer; turn the rest into findings with options | Engine |
| **3. Global invariants** [I3] | After every change, check the whole model; anything inconsistent becomes a finding, regardless of which operation caused it | Engine |
| **4. Resolve findings** [A4] | Choose among the options the engine offers, or propose a new typed operation | AI proposes, user confirms where approvals are affected |

The AI is never asked "what else needs to change?" The engine answers that. The AI is asked "here
are 5 findings, each with 2–3 options; which do you recommend and why?" It sees the findings and
the small subgraph around them, not the whole project. That keeps token use proportional to the
size of the decision, not the size of the project, and makes a missed consequence impossible to
hide: an unresolved finding blocks release.

## 3 Typed operations

Every model change that has a typed operation must use it, from the UI, the CLI or an AI agent
[I1]. An operation is a transaction:

1. **Validate preconditions** (object exists, target location exists, revision not released).
2. **Apply the direct edit.**
3. **Apply deterministic consequences**: changes with exactly one correct answer [I4].
4. **Run all invariants** (§4).
5. **Produce an impact report** [I2]: objects changed, outputs that will change (sheets, BOM lines,
   schedules, layout views, exports), findings opened and closed, approvals invalidated.
6. **Preview, then commit** to the working revision. Nothing is committed without the preview being
   available; an AI agent must show it to the user when approvals are invalidated.

Raw file edits are still possible (projects are Git repositories) but are followed by a full
invariant run, and anything inconsistent surfaces as findings.

### Initial operation catalog

| Operation | Direct edit | Deterministic consequences | Typical findings (decisions) |
|---|---|---|---|
| `move_device(dev, location)` | `dev.location` | Drop placement in old location; mark sheets/layout/BOM affected; invalidate approvals | Interconnects needed; power source; placement in new location; sheet grouping; cable entry; numbering; space |
| `rename_tag(dev, tag)` | `dev.tag` | None needed (all references are by ID) | Tag conflicts with scheme; released labels change |
| `replace_part(dev, part)` | `dev.part` | Remap pins with identical designations; update accessories required; mark geometry stale | Unmapped pins; rating changes; footprint no longer fits; new accessories |
| `delete_device(dev)` | Remove device | Remove its placements and sheet occurrences | Conductors left dangling; cable cores freed; I/O channel freed |
| `add_device(part, location)` | New device | Instantiate required accessories | Unplaced; undrawn; no supply scope |
| `connect(a, b, potential)` / `disconnect` | Conductor or core | Allocate wire number (new only) | Crosses locations [I7]; potential mismatch; terminal full |
| `reassign_channel(signal, channel)` | Move conductor end | Update derived I/O list and process-data mapping | Channel type mismatch; page strategy now splits a function |
| `move_to_sheet(occurrence, sheet)` | Sheet plan entry | Recompute cross-references | Deviation from page strategy needs a reason [D3] |
| `set_supply_scope(dev, scope)` | `dev.supply_scope` | BOM inclusion recomputed | Customer-supplied item with no interface definition |
| `change_convention(key, value)` | Project `profile/conventions.yaml` | Re-derive labels, section membership, I/O class labels for unreleased objects | Released labels or tags that would change [M10, C5]; objects now in the catch-all section [C6] |
| `promote_to_assembly(circuit)` | Library record | None in this project | Applicability conditions undefined |

New operations are added when a change type recurs. Each declares only its direct edit and its
single-answer consequences; detection of everything else is left to the invariants, so an
operation cannot "forget" a check.

### Worked example: `move_device(-K12, +CAB2)`

-K12 is a contactor whose coil is driven from a PLC output in +CAB1 and whose main contacts feed
motor -M3 in the field. After the direct edit (`location: loc_CAB2_MP`):

**Follows automatically** (derived or single-answer):

- BOM location column, per-cabinet device lists, I/O list location: derived, nothing to do.
- Contact cross-references on every sheet: derived.
- -K12's placement on +CAB1's mounting plate is removed; +CAB1's space use decreases.
- Approvals covering +CAB1 layout, +CAB2 layout and the affected sheets are invalidated.

**Findings with options** (more than one valid answer):

| Finding | Options the engine offers |
|---|---|
| F1: 3 conductors now join +CAB1 and +CAB2 directly [I7] (coil A1/A2 to PLC output and 0 V; aux 13/14 to an input) | (a) new interconnect cable +CAB1↔+CAB2 with terminals in both cabinets; engine sizes cores and allocates terminals; (b) use 3 spare cores of existing interconnect -W210 (if it has them) |
| F2: coil supply comes from +CAB1's 24 V | (a) keep it via the interconnect; (b) feed from +CAB2's PSU (engine shows +CAB2 PSU headroom change) |
| F3: -K12 is unplaced in +CAB2 [INV-6] | (a) auto-place in first free rail space; (b) leave for manual placement; shows +CAB2 occupancy change |
| F4: motor cable -W301 to -M3 now leaves from +CAB2 | (a) move its entry to +CAB2 (engine checks free Icotek positions); (b) route through +CAB1 terminals (adds an interconnect) |
| F5: sheet 12 groups +CAB1 power devices and still shows -K12 | (a) move the main-circuit occurrence to +CAB2's motor sheets; (b) keep it, recording the deviation [D3] |
| F6: circuit-group wire labels encode the area (CV2 `AAGGSS`), and -K12's coil and auxiliary circuits now sit in another cabinet's area | (a) renumber the affected wires; (b) keep the released numbers and record an exception [M10] |

The AI recommends an option for each (often the engine's default), the user confirms, the chosen
options are applied as further typed operations, and the invariants run again until no blocking
findings remain. Every step is in the change history, so the impact report doubles as the change
package for the next revision.

## 4 Invariants

Invariants are global, deterministic, and run after every change [I3]. Each has an ID, a severity
(`blocking` blocks release; `warning` does not) and a message that names the rule ID so the fix is
a lookup, as in LHMI's conformance gates.

| ID | Invariant | Severity |
|---|---|---|
| INV-1 | Every reference resolves (device, pin, potential, location, part version, sheet) | blocking |
| INV-2 | Every pin referenced exists on the part version | blocking |
| INV-3 | Every conductor has two ends; every cable core has two ends or is marked spare | blocking |
| INV-4 | No conductor joins two different locations; cross-location connections are cable cores or plug connections [I7] | blocking |
| INV-5 | Each conductor's ends are on its declared potential; no potential short through a terminal bridge | blocking |
| INV-6 | Every cabinet device with `supply_scope: by_us` or `by_customer` has a placement in its location's layout | blocking at release, warning before |
| INV-7 | Every device appears on at least one sheet, or is declared `not_drawn` with a reason (mechanical-only items) | blocking |
| INV-8 | Every sheet occurrence refers to an existing device and a valid symbol unit | blocking |
| INV-9 | Terminal occupancy within the part's conductor count and range per level | blocking |
| INV-10 | Each I/O channel has at most one assigned signal; signal type matches channel type | blocking |
| INV-11 | Required accessories of each part are instantiated | blocking |
| INV-12 | Every physical object has a `supply_scope` [I6] | blocking |
| INV-13 | Drawn/placed/procured reconciliation passes (§5) [I5] | blocking |
| INV-14 | Every external cable has a cable-entry assignment matching its diameter | blocking at release |
| INV-15 | Approvals reference the current hash of what they approve [W6] | blocking at release |
| INV-16 | Tags unique within their domain; numbering lock consistent with policy [M10] | warning (policy) |
| INV-17 | Each page declares a page strategy; deviations have a reason [D3] | warning |
| INV-18 | Each page belongs to exactly one package section [C6]; derived I/O class matches tag and section [CV1] | blocking |
| INV-19 | Labels and tags match the project's conventions (e.g. six-digit `AAGGSS` wire labels) or carry a recorded exception [C2, C4] | blocking |
| INV-20 | Numbers that labels and drawing IDs are built from are unique: RIO stations, FIO segments and cabinet areas do not collide, and none is `00` [CV2, CV4] | blocking |
| INV-21 | Drawing IDs are unique and locked; every off-page reference resolves to an existing drawing, sheet and zone [D8, DP6] | blocking |

Spares, intentionally unused pins, logical buses and shields have explicit semantics so they are
not reported as generic orphans.

## 5 BOM cross-reference: drawn ↔ placed ↔ procured

The requirement: **everything on the BOM is integrated, and everything integrated is on the BOM.**
Because BOM lines are derived from the model, most of this holds by construction. The gaps are in
the things that are *not* simply devices on a schematic: mechanical items, accessories, bulk
material, consumables, spares, customer-supplied items, and lines added by hand. LCAD makes the
check explicit with an expected-presence matrix.

### Object classes and expected presence

| Class | Drawn (schematic) | Placed (layout/3D) | Procured (BOM) |
|---|---|---|---|
| Cabinet electrical device (`by_us`) | required | required | required, counted once |
| Field device (`by_us`) | required | n/a (field) | required |
| Customer- or others-supplied device | required | as applicable | **must not** be on the procurement BOM; listed on the "by others" schedule |
| Existing device (retrofit) | required | as applicable | not procured; listed as existing |
| Mechanical-only item (enclosure, rail, duct, frame, gland plate, Icotek frame) | not required (`not_drawn`) | required | required |
| Required accessory (end stop, bridge, end plate, marker, blanking insert) | optional | derived from host | required via `required_by` link |
| Bulk material (wire, cable) | as conductors/cables | routes (estimate) | by type and length [B4] |
| Consumables and allowances (ties, ferrules, labels) | no | no | via a declared allowance rule |
| Spares | no | no | via the spare policy |
| Manual BOM line | — | — | allowed only with a justification and, where possible, a link to a model object |

### The checks

For every object, LCAD computes its actual presence (drawn / placed / procured) and compares it with
its class's expectation. For every BOM line, it computes what the line traces back to.

1. **Integrated but not procured:** a device drawn or placed, `by_us`, with no BOM line; a part with
   no orderable code (placeholder or unresolved); a required accessory not instantiated.
2. **Procured but not integrated:** a BOM line with no model object, allowance rule, spare policy or
   justified manual entry; a manual line whose linked object was deleted.
3. **Quantity mismatch:** BOM quantity ≠ count of installed instances (+ spares); length totals ≠
   sum of per-run estimates + allowances; accessory quantity ≠ host count × ratio.
4. **Presence mismatch:** placed in layout but not drawn (and not `not_drawn`); drawn but not placed
   (cabinet devices); customer-supplied item on the procurement BOM.

Each mismatch is a finding referencing [I5] and the object or BOM line. Unresolved mismatches block
release (INV-13). The report is a workbook sheet and a web view with three sections (1, 2, 3+4) so
a reviewer can work through it quickly.

### Example report rows

| # | Section | Object / line | Problem | Suggested fix |
|---|---|---|---|---|
| 1 | Integrated, not procured | -A5 in +CAB1 | Drawn and placed, but its part is still a placeholder envelope with no order code | Select a part under the sourcing policy (`replace_part`) |
| 2 | Integrated, not procured | -XT2 terminal strip | 2 end stops required by part, 0 instantiated | Add accessory (operation `add_device`) |
| 3 | Procured, not integrated | Manual line "DIN rail 35 mm, 2 m" | No rail object in any layout | Place rail in +CAB1 layout, or delete line, or justify |
| 4 | Quantity mismatch | wt_18AWG_BU | Manual override of 120 m; per-run estimates plus allowance total 105.6 m | Remove override, or justify it; reel rounding is shown separately |
| 5 | Presence mismatch | -B205 (by customer) | Appears on procurement BOM | Remove line; list on "by others" schedule |
