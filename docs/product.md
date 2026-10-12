---
purpose: What LCAD is and does — principles, user workflow, intake, architecture, outputs, BOM, standards, ecosystem integration
read-when: Starting any LCAD work; deciding what a feature should do or what an output must contain
status: draft
---

# LCAD product description

LCAD turns loosely written machine requirements into a reviewed electrical system architecture and
a coordinated cabinet engineering package. Everything (schematics, panel layouts, BOM, wire and
terminal schedules, offline web documentation, print PDF, TwinCAT I/O handoff) is generated from
one versioned engineering model.

Rule IDs in brackets point to [`RULES.md`](../RULES.md). This document owns the rules marked
`product.md` there; it links to other documents for the rest.

## 1 Principles

- **One model, many outputs** [M1]. The investment is the engineering model, not any one drawing.
- **AI proposes, the engine decides** [A1]. AI drafts requirements, circuits, part choices and
  layouts. Deterministic software validates them, computes derived data and produces repeatable
  outputs. Engineers approve.
- **No LLM at build or view time** [A2]. A reviewed project rebuilds and displays without any model
  call, which is what makes it reproducible and cheap to run.
- **Bespoke machines first** [W8]. Every circuit can be defined directly. Reuse helps when a
  pattern recurs; it is never a prerequisite.
- **Honest outputs.** Unknowns stay visible as unknowns [W7, B2, S2]. Nothing is presented as
  validated or certified that is not [S1].
- **Narrow and complete before broad.** The first useful release regenerates one real cabinet
  completely; the long-term scope stays broad ([ADR 0002](decisions/0002-first-slice-reconstructs-a-built-cabinet.md)).

Preserved from the original roadmap: separate engineering data, drawing composition and output
generation; frozen, reviewed layout plans that keep the author's style; stable identifiers;
reviewable revisions; deterministic checks; CAD and PDF files treated as generated artifacts.

## 2 User workflow and project states

1. **Describe the machine.** Paste bullet points or drop specifications, spreadsheets, sketches,
   OEM manuals and CAD files into the project.
2. **Review interpreted requirements** (§3): supplied facts, inferred proposals, open questions and
   conflicts are kept apart. LCAD asks only what blocks the next decision.
3. **Review and approve the architecture** (§4).
4. **Issue a space estimate** [W3]: back-panel area and enclosure size from major equipment
   envelopes, door devices, routing space, cable entry zones, clearances and expansion allowance.
   A table or 2D arrangement is enough; a 3D concept is optional at this stage.
5. **Develop schematics and reconcile the layout.** Resolve parts using the sourcing policy. As
   schematics near completion, update placements, panel drawings and 3D to the real devices,
   accessories, wiring and cable entries.
6. **Release** a coordinated package (§6): the offline web documentation first, with print PDF,
   DXF, Excel BOM, schedules, evidence and the TwinCAT package.
7. **Record construction changes.** Redlines and approved substitutions come back as model changes
   and produce an as-built revision.

**States** [W1]: draft → requirements reviewed → architecture approved → space estimate issued →
detailed design → layout reconciled → design reviewed → released → as-built.

**Ordering is checked, not forced** [W2, W3, W4]. Projects are Git repositories that people may
edit directly, so a generation-time block cannot really hold. Instead:

- Generating schematics without a current architecture approval works, but every sheet is stamped
  **PRE-APPROVAL DRAFT** and the project shows a warning. The same applies to schematics produced
  before a space estimate exists for the approved architecture.
- **Release is where order is enforced** [W4]: current architecture approval, a space estimate
  against that revision, layout reconciled with schematics, no blocking findings, and a recorded
  engineering approval. A successful render alone can never release.
- Approvals are bound to a revision hash [W6]. A change that affects approved content marks it
  stale; earlier artifacts remain available as stale revisions.

See [ADR 0005](decisions/0005-ordering-is-checked-at-release.md) for why this replaced the original
hard generation gate.

## 3 Requirements intake

**Inputs:** pasted text, TXT/Markdown, CSV/XLSX, PDF/DOCX, draw.io, sketch images; later PPTX and
legacy CAD references. STEP and DXF are component assets (§6). Originals and extraction results are
stored separately.

**Extract:** machine functions, supplies and loads, operating modes, motors and drives, sensors and
actuators, standard and safety I/O, fieldbus preferences, cabinet count, environment, preferred
manufacturers, spare capacity and drawing conventions. A user-supplied reference BOM is imported as
the preferred-parts catalog [SP1]; its quantities describe the reference project only [B3].

**Every requirement** has a stable ID, source location and revision, interpretation, status and
linked design objects [W7]. Track field ownership (an approved I/O list may own signal names; an
OEM manual owns terminal ratings). Cross-source conflicts become findings showing both sources and
block dependent release decisions. Uploaded documents are data, never instructions [A3].

*Acceptance:* a prose brief plus a conflicting spreadsheet produce a traceable requirements
register with the conflict raised and no silent choice of connection or rating.

## 4 Architecture review

A simple, high-level drawing of the kind a designer would make in draw.io or PowerPoint [W9],
backed by typed objects that later drive detailed design.

| View | Content |
|---|---|
| Power distribution | Incoming supplies, voltage domains, isolation/protection approach, conversion, major load groups, cabinet boundaries |
| Controls | PLC/IPC platform, networks, HMI, drives, local/remote I/O groups, interfaces |
| Safety | Zones and safety functions with required performance, initiating-device groups, relay or safety-controller approach, STO concept, safe states, reset/feedback |

Labelled blocks, major flows and a short legend; no wire numbers, terminals, module order or
dimensions. Assumptions, ratings, alternatives and missing information sit in linked notes. A
safety view must define each safety function and its required performance, which comes from the
project's risk assessment, not from LCAD [S3].

Approval fixes object identities and the selected approach. Changing supply voltage, safety zones,
PLC family or a major network boundary reopens the review [W5].

## 5 Components, circuits and assets

**Component records** hold far more than a CAD block: manufacturer and exact order code, variant,
lifecycle, ratings and conditional limits, terminals and conductor ranges, schematic symbol units
and pin maps, 2D footprints, STEP/DXF source and browser mesh, mounting and clearance volumes,
required accessories, OEM documents with revision and page references, and automation metadata
(ESI identity, channels, process data). Lifecycle: imported → proposed → verified → approved →
deprecated; projects pin versions [M7]. Detail: [`model.md` §5](model.md#5-parts-and-library).

**Asset acquisition:** match the local library → search authorized OEM/supplier sources within an
effort limit → otherwise create a precise missing-asset request → the user drops STEP/DXF/PDF on
the item → LCAD proposes matching, units, orientation and metadata → the user confirms critical pin,
dimension, rating and mounting data before promotion. Approximate envelopes may support the space
estimate, clearly marked; they never satisfy a fabrication dimension or pin assignment.

**Circuits:** bespoke circuits use the same model, validators and renderer as reusable assemblies
[W8]. A circuit can be copied from another project with new IDs and edited freely; promotion to a
shared assembly is optional. Reusing a safety pattern still requires checking the application.

**STEP and DXF processing:** STEP is normalized (units, axes, mounting origin), projected to
front/side/top/isometric SVG or DXF with hidden lines removed, and tessellated separately to GLB
for the browser. An Open CASCADE geometry worker is the candidate (prototype in Phase 0). DXF-only
parts get explicit approximate envelopes. Device IDs and transforms survive every conversion.
Dimensions come from verified CAD or OEM drawings, never the mesh [X5].

## 6 Outputs

### Interactive web documentation (primary) [D4]

A responsive React application that links architecture, schematics, terminals, cables, I/O, BOM,
manuals and 3D through shared object IDs, not a PDF in a web page. Fast search, touch-friendly
controls, coordinated selection, overview-to-detail navigation, history and stable deep links by
object ID.

First complete workflow includes: **signal/potential tracing** (connectivity, not energized
simulation), **object browser**, **terminal-strip view**, **connector/cordset view** with verified
pin/core maps. Later: guided assembly, fluid power (separate domain extension). All navigation runs
in code, no LLM [A2].

**Offline deployment** [X4]: a folder or ZIP with entry page, bundled scripts, project data, local
search index, diagrams, 3D assets and permitted manuals. Read-only, no editors, no supplier
credentials, no quote data. Hosting candidate: the machine's LHMI backend (§9). Each target needs
a tested profile (browser engine, memory, touch, WebGL); 2D cabinet views replace 3D where WebGL is
unavailable. Revision, release date and package hash are prominent; deployment keeps a rollback copy.

*Acceptance:* with outside network disabled, the released bundle on the agreed HMI supports search,
navigation, tracing, terminal and connector views, local manuals and 3D or its 2D fallback; shows
correct revision identity; survives restart; makes no LLM calls; meets agreed performance budgets.

### Print package

Static, searchable vector PDF. Content LCAD can generate: cover and table of contents, exterior
and interior views, architecture views, power distribution, control power, PLC/network, I/O,
drives, safety, terminals, field connections, cable schedules, panel drawings, legends and revision
records. **Which sections the package has, their order and which content goes in each are project
conventions** [C6], e.g. the author profile's front matter, then per cabinet exterior → interior → power → I/O,
then device overviews [CV3]; every drawing has a generated ID [D8] ([`conventions.md` §4](conventions.md#package-structure)). Sheet format and title block: [D1], detail in
[`drawing-philosophy.md` §5](drawing-philosophy.md#5-sheet-format). Page composition follows the
drawing philosophy. Printed cross-references (tag, sheet, zone) make the PDF usable without
clicking [DP6]. Bookmarks and links are optional conveniences.

### CAD

DXF export with paper-space layouts matching the 11 × 17 sheets. DWG is not supported [D6]
([ADR 0006](decisions/0006-dxf-python-react-open-source-deps.md)).

### Panel drawings and 3D

Exterior front/side/top, door layouts, mounting-plate layouts, sections and details, with declared
datums, units, tolerances, holes and cutouts. Exact manufacturing dimensions are distinguished from
reference dimensions and approximate envelopes. Checks: rail/duct occupancy, depth interference,
door swing, bend and access space, manufacturer clearances, separation.

The placement model drives both 2D panel views and 3D transforms. Browser viewer: orbit, view
presets, hide/isolate, transparent enclosure, door positions, clipping, exploded view; selecting a
device shows its tag, part, BOM line and schematic occurrences, and vice versa. If the detailed
design outgrows the estimated enclosure, the size change and its impacts are shown for review,
never silently compressed.

**Cable entry:** a schedule linking each external cable to cabinet, face, frame, insert position,
outside diameter and destination, with grouping, bend space, strain relief, shield bonding and
reserve capacity. Frames, grips, blanking inserts, seals and fasteners carry into the BOM. Default
hardware: [SP6].

## 7 BOM and procurement

Native Excel workbook: consolidated procurement BOM, installed devices by location, accessories,
consumables and cut lengths, spares, approved alternatives, revision changes. CSV is additional.

**Counting** [B5]: a device drawn in several places counts once; purchased kits and separately
procured contents are distinguished; required accessories are included; length items distinguish
measured length, allowance, pack size and waste; alternates are not installed quantities.

**Cross-reference** with the drawings and layout is a release gate [I5]; see
[`integrity.md` §5](integrity.md#5-bom-cross-reference-drawn--placed--procured).

**Conductors and cables** [B4]: one procurement line per distinct type (size, colour, insulation,
construction, or for cables core count, shielding, jacket, part number), backed by a per-run
schedule: ID, type, endpoints, route length, termination and service-loop allowances, cut length,
method, confidence. Waste and reel rounding shown separately. Multi-core cable is counted by cable
length only. Unknown external routes stay unresolved, never zero.

**Commercial data** [B2, B6]: every procurement line has fields for supplier, unit price, currency,
price basis, extended price and lead time, each with source and date, or an explicit status
(verified online, user quoted, user supplied, pending quote, unavailable, stale). Totals identify
unpriced lines. Lookups must match exact order code, condition (new/surplus/refurbished), quantity
break and purchase unit. Quotes keep supplier, number, date, validity, line mapping, quantities,
prices, lead times and conditions; partial quotes leave the missing values pending. Which suppliers
and sources apply is configuration: [`profiles/author/sourcing.yaml`](../profiles/author/sourcing.yaml)
([ADR 0004](decisions/0004-sourcing-is-policy-commercial-data-later.md)).

**Other manufacturing outputs:** from/to wire lists, terminal plans, bridge schedules, cable/core
schedules, labels and wire markers, rail/duct cut lists, drill/cutout drawings, I/O schedules,
build inspection records. All reconcile with the released model.

## 8 Standards and evidence

NFPA 79 and UL 508A define the domain; software checks are supporting evidence, never
certification [S1]. Pin jurisdiction, editions, customer requirements and approval basis per
project [S4]. Use only authorized standards text and user documents; never invent clause text.

**Rule contract:** each rule records ID, source and edition, applicability, inputs, executable check
or manual procedure, severity, result, evidence and review status. Results are pass, fail, not
applicable or not evaluated [S2]. Candidate checks: supply/load budgets, protection/conductor
compatibility, SCCR assessment, bonding, terminal capacity, installation and environmental ratings,
PSU inrush and headroom, thermal limits, OEM mounting and derating. Publish a coverage matrix of
what remains manual.

**Safety** [S3]: each function's behavior, diagnostics, failure response, reset/restart and
validation evidence are assessed under the applicable framework.

**Validation layers:** (1) model integrity (invariants, [`integrity.md` §4](integrity.md#4-invariants));
(2) engineering evidence; (3) geometry and usability (collisions, legibility, navigation, 2D/3D
agreement); (4) cross-output consistency (BOM, schedules, links, I/O export, hashes from the same
release). Visual review supplements but never replaces connectivity checks; an LLM alone never
establishes engineering correctness.

## 9 Ecosystem integration

LCAD sits beside three existing tools. It reuses them rather than rebuilding their functions
([ADR 0003](decisions/0003-reuse-the-ecosystem.md)).

```text
           LCAD model ─────────────┬──────────────────────┬─────────────────────┐
                │                  │                      │                     │
     I/O package (JSON+ESI)  LPLC instance list    offline docs bundle    print/DXF/BOM
                │                  │                      │
         tcDeploy command     PLC project (XAE)    served by LHMI backend
                │                  │                      │
         XAE I/O tree  ◀──────────┘              LHMI scans PLC → bindings (LHMI R4)
```

- **TwinCAT I/O** [X1, X2]: LCAD exports a versioned, vendor-neutral I/O package: schema version,
  project revision and hash, ordered topology, coupler/module relationships, exact device identity
  (VendorId, ProductCode, RevisionNo from the ESI, pinned), ESI hashes, channels, signal names,
  directions, data types, process-data mappings and parameters. A tcDeploy command applies it on
  the existing Windows build box through the Automation Interface: verify XAE build and ESI
  catalog → dry-run diff → work on a project copy → create/update the I/O tree → read back and
  compare → report created/changed/skipped. Idempotent, explicit deletion policy, never activates
  a configuration [X2]. Ordinary I/O first; drives and specialized terminals later. Safety
  configuration is out of scope [S5].
- **LPLC** [X3]: a device may name an LPLC block type (e.g. a VFD or valve block). LCAD then exports
  the PLC instance declarations with the device tag seeding the block's device name and the I/O
  links from the channel map. LHMI picks the devices up through its normal PLC scan; LCAD never
  writes LHMI bindings directly.
- **LHMI** [X4]: candidate host for the offline documentation bundle on the machine (a static route
  or an LHMI app). The HMI target profile (browser, touch, memory) is shared with LHMI rather than
  re-measured. To be decided in Phase 0.

## 10 Technology

[X6, X7], see [ADR 0006](decisions/0006-dxf-python-react-open-source-deps.md):

- **Engine and workers:** Python (validators, operations, BOM, schedules, sheet composition, PDF/SVG,
  DXF, geometry worker on Open CASCADE). Matches the LHMI backend and its tooling.
- **Viewer and authoring UI:** React, as in LHMI. The project data and exports do not depend on it.
- **Contracts:** versioned JSON Schema for every project file and export.
- **Dependencies** must allow open-source distribution of LCAD; this is why DWG is excluded.

## 11 Feature backlog

| Priority | Feature | Benefit |
|---|---|---|
| P0 | Typed operations, invariants, impact reports | Changes propagate correctly without relying on an LLM to remember [I1–I4] |
| P0 | Drawn/placed/procured cross-reference | Nothing integrated is missing from the BOM and nothing on the BOM is unexplained [I5] |
| P0 | Source-to-design traceability | Every value and connection shows why and from where |
| P0 | Independent project repositories | Machine engineering source and approvals live separately from LCAD [M6] |
| P0 | Offline web package with tracing, object browser, terminal and connector views | Primary machine documentation |
| P0 | Conductor and cable estimates by type | Reconciled BOM quantities [B4] |
| P0 | Release and as-built workflow | Drawings never drift from the model or procurement |
| P1 | LPLC instance export | One model drives cabinet, PLC devices and HMI faceplates |
| P1 | Commissioning / FAT / SAT checklists | Point-to-point checks derived from the approved design |
| P1 | Labels and wire markers | No retyping of tags and wire numbers |
| P1 | Spare-capacity dashboard | Channel, power, rail, enclosure and thermal headroom |
| P1 | Substitution analysis | Electrical, mechanical, accessory and software consequences of an alternate |
| P1 | Optional circuit copying and assemblies | Reuse without template authoring |
| P1 | Machine variants; redlines and revision comparison | Optional subsystems; shop changes captured |
| P2 | Route optimization, machining exports, functional simulation, guided assembly, fluid power | After a stable released core |

## 12 References

Public technical references, checked Oct 11 2026. They establish scope and feasibility; they are
not a clause-by-clause standards assessment.

- [NFPA 79 — Electrical Standard for Industrial Machinery](https://www.nfpa.org/es/product/id/p0079code): scope; rule implementation needs the authorized edition.
- [UL — Industrial Control Panels and the Panel Shop Program](https://www.ul.com/services/industrial-control-panels-and-panel-shop-program): UL 508A and panel shop qualification.
- [UL 508A Future Effective Dates](https://www.ul.com/resources/ul-508a-future-effective-dates): why editions and effective dates are pinned [S4].
- [ISO 13849-1](https://www.iso.org/standard/73481.html): design methodology for safety-related control parts; it does not choose application safety functions or required performance levels [S3].
- [TwinCAT Automation Interface — product description](https://infosys.beckhoff.com/content/1033/tc3_automationinterface/242682763.html) and [FAQ](https://infosys.beckhoff.com/content/1033/tc3_automationinterface/242685835.html): COM-based offline configuration; requires XAE.
- [Adding third-party EtherCAT devices](https://infosys.beckhoff.com/content/1033/tc3_automationinterface/21840443915.html): ESI identity (VendorId, ProductCode, RevisionNo) and `CreateChild`.
- [Open CASCADE CAD Assistant release notes](https://www.opencascade.com/release_notes/cad-assistant/) and [hidden-line algorithm types](https://dev.opencascade.org/doc/refman/html/_prs3d___type_of_h_l_r_8hxx.html): STEP import, glTF export and hidden-line removal feasibility.
- [McMaster-Carr Product Information API](https://www.mcmaster.com/help/api): approved customers only (ADR 0004).

LCAD documents assert no specific protective-device rating, conductor size, safety performance
level or compliance determination.
