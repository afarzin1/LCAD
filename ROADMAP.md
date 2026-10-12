---
purpose: Phases, what is in scope now, what is implemented, and the open decisions
read-when: Planning or finishing work; update in the same change when status changes
status: draft
---

# LCAD roadmap

**Implemented:** nothing yet. LCAD is at the design stage (rule P5).

The order below is the **build order**. It is not the order a user follows in a project; that is
in [`docs/product.md` §2](docs/product.md#2-user-workflow-and-project-states). The first milestones
regenerate a cabinet that has already been built, so every output has a ground truth
([ADR 0002](docs/decisions/0002-first-slice-reconstructs-a-built-cabinet.md)). No schedule is
promised before Phase 0 has measured the reference cabinet, library coverage and the integration
proofs.

## Phase 0 — Reference cabinet, schema v0, technical proofs

- Choose one built and documented cabinet (drawings, BOM, I/O list, manuals, some STEP files).
  **Candidate:** the main control panel of the author's private reference package
  ([`examples/reference-package-notes.md`](examples/reference-package-notes.md)): safety I/O, RIO and FIO, power distribution, a
  typical, and a second cabinet for moves. Its model is written in `examples/private/` until it is
  de-identified (P8).
  Still needed: its BOM document and I/O list, and STEP files for the iso views.
  Prefer one with a safety relay, at least two I/O modules and power distribution, so DP3–DP5 are
  exercised, and ideally a second cabinet or field area so moves between locations can be tested.
- Write it by hand in the schema ([`docs/model.md`](docs/model.md)); revise the schema until it fits.
- From the example package, complete the author profile (`profiles/author/conventions.yaml`):
  tag scheme, sheet numbering, wire label details, I/O class naming.
- Collect drawing-philosophy examples and answer the questions in
  [`docs/drawing-philosophy.md` §6](docs/drawing-philosophy.md#6-open-questions-for-the-user);
  measure the reference style (text heights, rung pitch, title block).
- Proofs: one I/O module page rendered to SVG and an 11 × 17 PDF; one STEP part to isometric SVG and
  GLB (Open CASCADE Python bindings); one I/O package applied by a tcDeploy prototype command to an
  offline XAE project; the offline docs host decision (LHMI static route or app) and target profile.

**Exit:** reference cabinet fully encoded; schema v0 accepted; each proof works or its limits are
recorded; open decisions below answered or deliberately deferred.

## Phase 1 — Reconstruction: deterministic core

- Invariants INV-1 to INV-17 ([`docs/integrity.md` §4](docs/integrity.md#4-invariants)).
- Typed operations: `move_device`, `rename_tag`, `replace_part`, `delete_device`, `add_device`,
  `connect`/`disconnect`, `reassign_channel`, with impact reports.
- Derived outputs: BOM workbook (engineering columns; commercial fields present but empty with
  status), I/O list, wire list, terminal plan, cable/core schedule.
- Drawn/placed/procured cross-reference report [I5].
- Sheet planner and renderer for PS-2 (I/O module) and PS-4 (power distribution) pages, SVG and PDF,
  11 × 17 with title block.
- Numbering allocator and lock file.
- Conventions engine: project `profile/` copied from a profile; package sections; derived I/O
  classes; `change_convention`; six-digit wire-label allocator (ADR 0009); drawing-ID
  allocator and drawing/sheet.zone cross-references (ADR 0010); INV-18 to INV-21.

**Exit:** regenerated I/O pages, power pages, BOM, I/O list, wire list and terminal plan reconcile
with the reference package (differences explained). `move_device` between two locations produces the
correct findings and, once resolved, a clean invariant run and a correct change package.

## Phase 2 — Review viewer and remaining drawing types

- PS-1 device pages with isometric views (DP1, DP2) and PS-3 function pages for the safety relay
  (DP4); cross-references (DP6).
- Offline React viewer: search, object browser, tracing, terminal-strip and connector views, deep
  links; packaged for the Phase 0 host.
- Panel 2D layout and space estimate; DXF export.
- Component asset intake (STEP/DXF drop, confirm critical data).

**Exit:** the complete reference schematic set regenerates in the user's style; the offline bundle
passes its acceptance test on the target ([`docs/product.md` §6](docs/product.md#6-outputs)).

## Phase 3 — Intake, architecture and approvals

- AI intake to a requirements register with sources, statuses and conflict findings (W7).
- Architecture objects and the three views (W9); approvals bound to hashes (W6); draft stamps and
  release checks (W2–W4).
- AI proposals expressed as typed operations; findings resolution loop (A4).

**Exit:** a real brief becomes a reviewed architecture; a project carried from brief to release
shows the required order in its history; a change reopens the right approvals.

## Phase 4 — Ecosystem and fabrication package

- TwinCAT I/O package and tcDeploy apply command (X1, X2); LPLC instance export (X3).
- Browser 3D, reconciliation of space estimate with final layout, dimensioned panel drawings.
- Cable entry schedule with Icotek defaults (SP6); conductor and cable estimates by type (B4).
- Labels and wire markers; standards rule packs with coverage matrix (S1–S4).

**Exit:** a complete cabinet package passes a real build review with no undisclosed unsupported
content; one device is traceable from requirement to architecture, wiring, BOM, 2D/3D placement,
I/O export and PLC instance.

## Phase 5 — Commercial data and lifecycle

- Sourcing policy engine (B1, SP1–SP6); DigiKey/Mouser API lookups; McMaster API if approved;
  quote ingestion; commercial snapshots and statuses (B2, B6).
- As-built workflow and redlines; variants; substitution analysis; commissioning/FAT/SAT records;
  machining exports; specialized TwinCAT devices.

**Exit:** a part substitution produces a correct change package without altering unaffected
approved work; a BOM carries complete price and lead-time provenance or explicit pending statuses.

## Open decisions

| Decision | Needed by | Notes |
|---|---|---|
| Reference cabinet | Phase 0 start | Candidate: main control panel of the private reference package |
| Tag scheme and wire-label details (per potential or per conductor, start, width) | Phase 0 | From the example package; author profile CV2 |
| Drawing-philosophy rulings (DP1 scope, T1–T3) | Phase 1 sheet planner | With the user's examples |
| Offline docs host: LHMI static route vs LHMI app | Phase 0 | ADR 0003 |
| First TwinCAT/XAE build and I/O families | Phase 0 proof | tcDeploy targets 4026 |
| Jurisdiction, standards editions, licensed access | Phase 4 | S4 |
| Who gives final engineering approval | Phase 3 | W4 |
| Permitted contents of the HMI package | Phase 2 | Manuals, prices excluded by default |
