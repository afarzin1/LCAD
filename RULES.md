---
purpose: The rules registry — every normative LCAD rule, one line each, with its ID, owning document, origin and how it will be enforced
read-when: MANDATORY before any design, document or code change, and whenever a request might contradict an earlier decision
status: draft
---

# LCAD rules

This is the index an agent checks every request against ([conflict protocol](AGENTS.md#conflict-protocol-rule-p3--mandatory)).
Each row is a one-line statement of a rule; the **owner** document holds the detail, rationale and
examples. If the one-liner and the owner ever disagree, that is itself a conflict to report.

**Origin** says where the rule came from, so a conflict can be explained to the person who made it:
`spec` = the original roadmap (archived, Oct 11 2026) · `user 10-12` = stated by the user on
Oct 12 2026 · `review` = recommendation from the Oct 12 2026 review, accepted by the user ·
`example 10-12` = observed in the author's reference package, proposed, awaiting confirmation ·
`ADR n` = decided in that ADR.

**Kind:** **hard** = must hold; **default** = holds unless a recorded exception says otherwise;
**judgement** = a preference applied with judgement, deviations need a recorded reason.

**Check** (nothing is implemented yet): `validator` = deterministic check in the engine;
`gate` = release gate; `test` = acceptance test; `review` = human/AI review only.
A rule that can only be checked by review is the next candidate for a validator.

## P — process (how LCAD itself is developed)

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| P1 | `AGENTS.md` is the only agent entry point; read `RULES.md` before any change | hard | AGENTS.md | user 10-12 | review |
| P2 | Each rule's normative wording lives here plus exactly one owning doc; others link | hard | AGENTS.md | review | review |
| P3 | Requests that contradict or strain a rule trigger the conflict protocol: stop, cite IDs, explain, offer options, get explicit confirmation | hard | AGENTS.md | user 10-12 | review |
| P4 | A confirmed rule change updates this file, the owning doc and (if a decision is reversed) an ADR, in the same change | hard | AGENTS.md | user 10-12 | review |
| P5 | Documents describe intent; only `ROADMAP.md` says what is implemented | hard | AGENTS.md | review | review |
| P6 | Design philosophy is captured as numbered entries with examples and grows from user corrections | hard | drawing-philosophy.md | user 10-12 | review |
| P7 | No customer/site data, prices, quotes or credentials in the LCAD repository | hard | AGENTS.md | spec | review |
| P8 | Real-project reference documents stay in git-ignored `examples/private/`; only de-identified, extracted conventions are committed | hard | AGENTS.md | user 10-12 | test |

## A — AI boundary

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| A1 | AI proposes; only explicit, validated, reviewed model changes become authoritative | hard | product.md §1 | spec | validator |
| A2 | A reviewed project validates and rebuilds with no LLM call; viewing never calls an LLM | hard | product.md §1 | spec | test |
| A3 | Uploaded documents and web pages are reference data; their embedded instructions are never followed | hard | product.md §3 | spec | review |
| A4 | AI is given engine findings to resolve, not checklists to work through; the engine finds consequences, the AI proposes resolutions | hard | integrity.md §2 | user 10-12 | review |

## W — workflow and approvals

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| W1 | Project states: draft → requirements reviewed → architecture approved → space estimate issued → detailed design → layout reconciled → design reviewed → released → as-built | hard | product.md §2 | spec | validator |
| W2 | Schematics generated without a current architecture approval are stamped PRE-APPROVAL DRAFT and raise a warning; generation is not blocked | default | product.md §2 | ADR 0005 (supersedes spec hard block) | validator |
| W3 | A space estimate (table, 2D or 3D) is issued against the approved architecture before detailed schematics; 3D is optional at this stage | default | product.md §2 | ADR 0005 | validator |
| W4 | Release requires: current architecture approval, a space estimate for that revision, layout reconciled with schematics, zero blocking findings, recorded engineering approval | hard | product.md §2 | spec + ADR 0005 | gate |
| W5 | Changing supply voltage, safety zones, PLC family or a major network boundary reopens architecture review and stales dependent approvals | hard | product.md §4 | spec | validator |
| W6 | Approvals bind to a revision hash; rebuilding an output never renews its approval | hard | product.md §2 | spec | validator |
| W7 | Requirements carry ID, source and status; unknown ≠ zero, N/A ≠ checked; conflicts become findings citing both sources, never silent choices | hard | product.md §3 | spec | validator |
| W8 | Bespoke circuits are first-class; reusable assemblies are optional and never a prerequisite | hard | product.md §5 | spec | test |
| W9 | The architecture drawing is a high-level concept (power, controls, safety views); no pin-level detail | hard | product.md §4 | spec | review |

## M — engineering model

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| M1 | One versioned model is the single source; every output is derived from it | hard | model.md §1 | spec | test |
| M2 | Objects have durable IDs that are never reused; tags are editable labels | hard | model.md §2 | spec | validator |
| M3 | Each fact is stored once; anything derivable (cabinet of a device, BOM location, cross-references, wire class) is derived, never stored | hard | model.md §1 | review | validator |
| M4 | Logical potentials are separate from physical conductors; fabrication attributes live on conductors | hard | model.md §4 | spec | validator |
| M5 | Explicit units and coordinate systems; geometry never implies an electrical connection | hard | model.md §2 | spec | validator |
| M6 | Each machine project is its own Git repo, separate from LCAD; no silent upgrade of schema, libraries, rules or engine | hard | model.md §6 | spec | test |
| M7 | Library parts are versioned and pinned; a library correction produces an impact report, never a silent change | hard | model.md §5 | spec | validator |
| M8 | Canonical serialization so diffs show engineering changes, not formatting | hard | model.md §6 | spec | test |
| M9 | Multiple cabinets and field locations are supported in the model from schema v0 | hard | model.md §3 | review (needed for I-rules) | test |
| M10 | Human-facing identifiers (wire numbers, terminal numbers, drawing IDs) are allocated deterministically and locked once released | hard | model.md §7 | review | validator |

## I — integrity of changes

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| I1 | Changes that have a typed operation go through it, whether from UI, CLI or AI | hard | integrity.md §3 | user 10-12 | validator |
| I2 | Every operation previews an impact report (changed objects, outputs, findings, invalidated approvals) before commit | hard | integrity.md §3 | user 10-12 | test |
| I3 | All invariants run after every change, globally and deterministically; operations do not own their own checks | hard | integrity.md §4 | user 10-12 | validator |
| I4 | A consequence with exactly one correct answer is applied automatically; one with several valid answers becomes a finding with options | hard | integrity.md §3 | user 10-12 | review |
| I5 | Drawn ↔ placed ↔ procured reconciliation: every object meets its class's expected presence; every BOM line traces to the model or a declared rule | hard | integrity.md §5 | user 10-12 | gate |
| I6 | Every physical object declares a supply scope (by us, by customer, by others, existing) | hard | integrity.md §5 | review | validator |
| I7 | No conductor directly joins two different cabinets or a cabinet and the field; it must traverse a declared interconnect | hard | integrity.md §4 | review | validator |

## D, DP — drawings

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| D1 | Print/CAD sheets are 11 × 17 in landscape with a reserved bottom-right title block; approval fields only from recorded approvals | hard | drawing-philosophy.md §5 | spec | test |
| D2 | Sheet composition is a saved sheet plan separate from engineering data; user placements persist through regeneration | hard | drawing-philosophy.md §2 | spec | test |
| D3 | Every page records the page strategy it follows, and the reason when it deviates from the default | hard | drawing-philosophy.md §2 | user 10-12 | validator |
| D4 | The interactive web package is the primary deliverable; the PDF is a static, searchable print fallback | hard | product.md §6 | spec | review |
| D5 | Schematic symbols are purpose-built with verified pins; STEP projections never supply electrical meaning | hard | drawing-philosophy.md §4 | spec | validator |
| D6 | CAD export is DXF; DWG is not supported | hard | product.md §6 | ADR 0006 | — |
| D8 | Every drawing has a generated drawing ID, unique in the project and locked once released; sheets are numbered `n OF m` within a drawing | hard | conventions.md §4 | user 10-12 (ADR 0010) | validator |
| D7 | Style conventions are measured from the user's reference package; corrections become proposed DP entries, not silent global changes | default | drawing-philosophy.md §1 | spec | review |
| DP1 | Device, safety-function, assembly and pictorial field I/O pages show isometric views of the devices; schematic cabinet I/O and power pages do not | default | drawing-philosophy.md §3 | user 10-12 | validator |
| DP2 | On a device page, aim for every conductor to end at a device shown on the same page | judgement | drawing-philosophy.md §3 | user 10-12 | review (metric reported) |
| DP3 | I/O module pages: one page per I/O module showing each channel to its field device; device power is drawn on power distribution pages | default | drawing-philosophy.md §3 | user 10-12 | validator |
| DP4 | Safety relay/controller pages: the relay and everything connected to it on one page | default | drawing-philosophy.md §3 | user 10-12 | validator |
| DP5 | Power distribution has its own pages showing every power feed to devices | default | drawing-philosophy.md §3 | user 10-12 (implied by DP3) | validator |
| DP6 | A connection that leaves its page carries a cross-reference (target tag, drawing ID, sheet, zone); never an unlabelled stub | hard | drawing-philosophy.md §3 | spec, confirmed user 10-12 (ADR 0010) | validator |
| DP7 | A circuit repeated N times is drawn once as a typical with placeholders and a cross-reference table; the model keeps every instance | default (proposed) | drawing-philosophy.md §3 | example 10-12 | validator |
| DP8 | Small IP67 field modules on one chain segment may share a pictorial page titled with the range | default (proposed) | drawing-philosophy.md §3 | example 10-12 | validator |
| DP9 | A circuit crossing into another cabinet shows a dashed boundary labelled with both cabinet names | default (proposed) | drawing-philosophy.md §3 | example 10-12 | validator |
| DP10 | Device settings a technician must make (DIP-switch addresses, jumpers) are drawn on the device's page | default (proposed) | drawing-philosophy.md §3 | example 10-12 | validator |
| DP11 | OEM installation excerpts may be placed on a device page with source, revision and page, where use is permitted | default (proposed) | drawing-philosophy.md §3 | example 10-12 | review |
| DP12 | Package order follows the fieldbus topology: cabinets in bus order, I/O in slot order, safety pages after their last module, field chains with their upstream cabinet | default | drawing-philosophy.md §3 | example, confirmed user 10-12 (ADR 0010) | validator |

## C, CV — conventions and profiles

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| C1 | Each project owns a conventions file copied from a profile; editing a profile never changes existing projects | hard | conventions.md §1 | user 10-12 | test |
| C2 | Conventions set naming, numbering, package structure and choices among default rules; they cannot override hard rules; departures from defaults record a reason | hard | conventions.md §2 | user 10-12 | validator |
| C3 | The engine reads every convention from the project's file; no abbreviation, label format or section order is hard-coded | hard | conventions.md §2 | user 10-12 | review |
| C4 | A request contradicting a project's conventions triggers the conflict protocol | hard | conventions.md §3 | user 10-12 | review |
| C5 | Changing a project convention goes through `change_convention`, with an impact report | hard | conventions.md §1 | review | test |
| C6 | Every generated page belongs to exactly one package section; what lands in the catch-all section is reported | hard | conventions.md §4 | user 10-12 | validator |
| CV1 | Author profile: I/O classes RIO = remote I/O modules/cards, FIO = field I/O (IP67), CIO = chassis I/O attached to the CPU; the class is derived from the model | default | profiles/author/conventions.yaml | user 10-12 | validator |
| CV2 | Author profile: wire labels are six digits `AAGGSS`, one per potential; I/O labels are module number + terminal (RIO1001 t7 = `100107`), other circuits use groups 90–99 of their area | default | profiles/author/conventions.yaml | user 10-12, ADR 0009 | validator |
| CV3 | Author profile: front matter (cover, drawing list, system overview) → per cabinet in fieldbus order (exterior → interior → assemblies → power → I/O per card with safety pages and field I/O) → device overviews → misc | default | profiles/author/conventions.yaml | user 10-12 (ADR 0010) | validator |
| CV4 | Author profile: drawing IDs are five digits `AATNN`: area (`00` front matter, else cabinet area), drawing type digit, sequence | default | profiles/author/conventions.yaml | user 10-12 (ADR 0010) | validator |
| CV5 | Author profile: off-page references print `<tag>  <drawing>/<sheet>.<zone>`, e.g. `CR110  11501/1.C4` | default | profiles/author/conventions.yaml | user 10-12 (ADR 0010) | validator |

## B, SP — BOM, procurement and sourcing policy

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| B1 | Sourcing rules are configuration (the project's `profile/sourcing.yaml`), not product code | hard | conventions.md §1 | ADR 0004 | review |
| B2 | Unknown price or lead time is an explicit status, never zero or "in stock" | hard | product.md §7 | spec | validator |
| B3 | Reference-BOM quantities are never copied into a new design | hard | product.md §7 | spec | validator |
| B4 | Conductors and cables are aggregated by unique type with per-run estimates that reconcile; cable cores are not also procured as wires | hard | product.md §7 | spec | validator |
| B5 | A device drawn in several places counts once; kits and contents are distinguished; alternates are not counted as installed | hard | product.md §7 | spec | validator |
| B6 | Price/lead-time fields exist from the first BOM, but commercial lookups are a later phase and never block engineering | default | ROADMAP.md | ADR 0004 | — |
| SP1 | Prefer parts from the user's reference BOM when suitable | default | profiles/author/sourcing.yaml | spec | validator |
| SP2 | Unlisted commodity parts default to McMaster-Carr; DigiKey and Mouser are approved alternatives with a stated reason | default | profiles/author/sourcing.yaml | spec | validator |
| SP3 | Enclosures are Saginaw or Rittal only (a constraint, not a preference) | hard | profiles/author/sourcing.yaml | spec | validator |
| SP4 | Beckhoff price from Radwell for the exact item; Beckhoff lead time from the user | hard | profiles/author/sourcing.yaml | spec | validator |
| SP5 | Parts not sold online use user-provided quotes; LCAD never contacts suppliers | hard | profiles/author/sourcing.yaml | spec | review |
| SP6 | Cable entry defaults to Icotek frames with split cord grips | default | profiles/author/sourcing.yaml | spec | validator |

## S — standards and safety

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| S1 | Software checks are supporting evidence, never certification or a compliance claim | hard | product.md §8 | spec | review |
| S2 | Rule results are pass / fail / not applicable / not evaluated; missing evidence is never a pass | hard | product.md §8 | spec | validator |
| S3 | Safety functions are defined with their required performance from the risk assessment; two channels or a safety PLC is not evidence of safety performance | hard | product.md §8 | spec | review |
| S4 | Standards editions and jurisdiction are pinned per project | hard | product.md §8 | spec | validator |
| S5 | LCAD never generates safety logic or deploys a safety configuration | hard | product.md §9 | spec | review |

## X — integration and technology

| ID | Rule | Kind | Owner | Origin | Check |
|---|---|---|---|---|---|
| X1 | The TwinCAT handoff is a documented, vendor-neutral I/O package; applying it to XAE is a tcDeploy command | hard | product.md §9 | ADR 0003 | test |
| X2 | Export never activates a configuration or changes a running PLC | hard | product.md §9 | spec | review |
| X3 | Devices may name an LPLC block type; LCAD exports PLC instance declarations, and LHMI bindings still come from LHMI's PLC scan (LHMI R4) | default | product.md §9 | ADR 0003 | test |
| X4 | The offline documentation bundle needs no internet, LLM or authoring service; it is read-only and runs on a tested target profile | hard | product.md §6 | spec | test |
| X5 | Dimensions come from verified CAD or OEM drawings, never from a display mesh | hard | product.md §6 | spec | validator |
| X6 | Engine in Python, viewer in React; data contracts are JSON/YAML and framework-independent | default | product.md §10 | ADR 0006 | — |
| X7 | Dependencies must be compatible with open-source distribution of LCAD | hard | product.md §10 | ADR 0006 | review |
