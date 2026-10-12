---
purpose: The user's design philosophy for electrical drawings — page strategies, what goes on which page, device views, sheet format — as numbered entries that grow from examples and corrections
read-when: Composing or reviewing any drawing sheet; changing the sheet planner or renderer; whenever the user corrects how a drawing was laid out
status: draft (seeded Oct 12 2026; evidence added from the author's reference package the same day)
---

# Drawing philosophy

Good schematics are mostly about **what goes on which page**. That is a judgement the user makes
well and an algorithm makes badly unless the judgement is written down. This document records it
as numbered entries (`DP*`), each with its reasoning, when it applies, when it does not, and
real examples. It grows as the user supplies examples and corrects drawings [P6].

## 1 How this document works

**Entry format.** Every entry has:

- **Statement**: one sentence, mirrored in [`RULES.md`](../RULES.md).
- **Kind**: *default* (holds unless an exception is recorded) or *judgement* (a goal; a deviation
  needs a reason).
- **Why**: the reasoning, in the user's terms.
- **Applies when / does not apply when**: the boundary, as concretely as possible.
- **Wins over / yields to**: how it ranks against other entries when both apply.
- **Examples**: files in [`examples/drawing-philosophy/`](../examples/drawing-philosophy/) with a
  one-line note of what each shows.
- **Origin**: who stated it and when.

**How entries grow.** When the user corrects a drawing ("put the PSU feed on the power page, not
here"), the agent does not just fix that sheet. It asks whether the correction is:

1. a one-off for this sheet (recorded as a sheet-plan exception with the reason),
2. a project convention (recorded in the project's `profile/conventions.yaml`, see
   [`conventions.md`](conventions.md)), or
3. a general philosophy (a new or amended `DP` entry here, via the conflict protocol if it changes
   an existing entry) [D7].

Style measurements from the user's reference package (text heights, rung pitch, margins, line
weights, symbol spacing) are recorded in §5 as they are measured.

## 2 Sheet plan and page strategies

Sheet composition is a saved artifact, `layout/sheets.yaml`, separate from the engineering model
[D2]. It lists the pages, the **page strategy** each page follows, which device occurrences and
symbol units are on it, and any user placements, which persist through regeneration.

```yaml
# layout/sheets.yaml (excerpt)
- sheet: "20"
  title: "Safety relay -K40"
  strategy: function_page            # PS-3
  anchor: dev_K40                    # the device or function the page is built around
  occurrences: [dev_K40, dev_S40, dev_S41, dev_B42, dev_K41, dev_K42]
- sheet: "31"
  title: "Digital inputs -A31"
  strategy: io_module_page           # PS-2
  anchor: dev_A31
  deviations:
    - { what: "-B205 power shown here", reason: "single loop-powered sensor; no PD page for it", by: user, date: 2026-10-12 }
```

Pages are grouped into the **package sections** defined by the project's conventions [C6]; with the
author profile: front matter → per cabinet in fieldbus order (exterior → interior → assemblies →
power distribution → I/O pages with their safety pages and field I/O) → device overviews → misc
[CV3]. Every drawing gets a generated ID (`10501`) [D8, CV4]. Each page records its strategy and the reason for
any deviation [D3]. The engine reports how well
each page meets its strategy (e.g. the share of a device's connections that end on the same page),
so drift is visible.

### Page strategies

| ID | Strategy | Built around | Typical use |
|---|---|---|---|
| PS-1 | **Device page** | One device and everything it connects to | Complex standalone devices: a drive, a valve terminal, a smart instrument |
| PS-2 | **I/O module page** | One I/O module (or a small group) | Channels to field devices [DP3]. Two variants: *schematic* for cabinet I/O (RIO), *pictorial* for plug-in field I/O (FIO) [DP1] |
| PS-3 | **Function page** | One function, e.g. a safety relay and its loop | Safety relays and controllers [DP4]; other self-contained functions |
| PS-4 | **Power distribution page** | A supply and its feeds | Every power feed to devices [DP5] |
| PS-5 | **Overview / index pages** | The package | Cover and drawing list, control system (network) overview, legends |
| PS-6 | **Typical page** | A circuit repeated N times | Drawn once with placeholders and a cross-reference table [DP7] |
| PS-7 | **Assembly page** | A mechanical/electrical assembly | Exploded isometric with callouts, e.g. a harness and its connector inserts |

### Choosing a strategy: engine first, judgement second

1. **The engine proposes** a plan from the model using the entries below: hubs (I/O modules, PSUs,
   distribution) get their own pages; safety functions are kept whole; field devices appear on
   their I/O page with power on power distribution pages.
2. **The engine measures** each candidate: connections ending off-page, page density against the
   sheet's usable area, whether any safety function is split, number of cross-references.
3. **The AI chooses where the metrics are close** and writes the reason into the sheet plan. This is
   where judgement is spent, and only there.
4. **The user reviews.** Corrections follow §1.

## 3 Entries

### DP1 — Device wiring pages show an isometric view of the device

- **Kind:** default · **Origin:** user, Oct 12 2026
- **Why:** the person wiring the device sees what it physically looks like and where the terminals
  are, next to the schematic.
- **Applies when:** a page is built around a device (PS-1) or a function anchored on a device
  (PS-3), and a verified STEP model exists.
- **Fallback:** without STEP, use the OEM front view or a dimensioned outline, labelled as such; an
  approximate envelope is never presented as the real device.
- **Also applies to:** pictorial field I/O pages (PS-2 pictorial), where the IP67 module and every
  device are shown as isometric views joined by named cables, and assembly pages (PS-7).
- **Does not apply to:** schematic I/O pages for cabinet I/O (RIO) and power distribution pages,
  which use schematic symbols. An OEM terminal or contact diagram image may be added next to a
  device there when it helps the wirer.
- **Examples:** [reference package](../examples/reference-package-notes.md): the safety gate-lock page, the pictorial field I/O pages and the harness
  page use iso views; the cabinet I/O pages do not (one adds an OEM contact diagram of an e-stop).

### DP2 — On a device page, aim for every conductor to end at a device shown on the same page

- **Kind:** judgement · **Origin:** user, Oct 12 2026
- **Why:** a technician can trace a device's wiring without flipping pages.
- **Applies when:** PS-1 and PS-3 pages.
- **Yields to:** DP3 and DP5 when the device is a field device on an I/O module page. Its power is
  then on a power distribution page by design, not a failure of DP2.
- **Measured:** the engine reports the share of the anchor's connections that end on the page.
- **Examples:** [reference package](../examples/reference-package-notes.md): on the gate-lock page every conductor ends on the page, including channels
  from two different safety modules; on field I/O pages every device sits on its port.

### DP3 — One page per I/O module; device power on power distribution pages

- **Kind:** default · **Origin:** user, Oct 12 2026 ("very frequently")
- **Statement:** an I/O module gets its own page showing each channel to its field device. Power to
  those field devices is drawn on the power distribution pages, with cross-references.
- **Why:** to be confirmed by the user. Likely: the I/O page reads as the I/O list, and power is
  easier to check for budget and protection when it is all in one place.
- **Wins over:** DP2 for field devices.
- **Yields to:** DP4 for a safety input or output that is part of a safety function page: the I/O
  page keeps the channel and a "see overview" note; the wiring is drawn on the function page (T1).
- **Grouping:** small field modules (e.g. 4-channel IP67 boxes) may share a page, titled with the
  range (`FIO2101-FIO2108`) [DP8].
- **Examples:** [reference package](../examples/reference-package-notes.md): I/O pages note "SEE PWR DISTRIBUTION DRAWING"; safety channels note "SEE
  SAFETY GATE LOCK OVERVIEW DRAWING"; field modules are grouped up to eight per page.

### DP4 — A safety relay and everything connected to it on one page

- **Kind:** default · **Origin:** user, Oct 12 2026 (example given)
- **Why:** the whole safety function (inputs, relay, outputs, reset, feedback) can be read and
  reviewed in one place.
- **Applies when:** a safety relay or a small safety controller with a bounded loop.
- **Also applies to:** a safety function built on safety I/O modules (TwinSAFE): the function page
  collects the channels from every module involved, the safety device with its iso view, and the
  cable with its cores.
- **Placement:** inline in the I/O section, right after the last module the function uses [DP12].
- **Does not apply when:** to be confirmed. Probably a large safety PLC with many functions, where
  each function may be a page instead.
- **Examples:** [reference package](../examples/reference-package-notes.md): a gate-lock page combining a safety input module, a safety output module
  and the lock; a gate-control page.

### DP5 — Power distribution has its own pages showing every feed

- **Kind:** default · **Origin:** user, Oct 12 2026 (implied by DP3)
- **Why:** supply budgets, protection and feeds are checked in one place.
- **Shape:** per cabinet, from the incoming feed through the electronic breaker channels to every
  device's power terminals; daisy-chained supplies drawn as jumpers; PE drops shown; multi-sheet
  as needed (`1 OF 2`).
- **Examples:** [reference package](../examples/reference-package-notes.md): 24 V DC distribution over two sheets; a high-voltage DC bus with current
  transformers.

### DP6 — Every connection that leaves a page carries a cross-reference

- **Kind:** hard · **Origin:** original spec
- **Statement:** a connection that continues on another page shows the target tag, sheet and zone.
  Never an unlabelled stub. Coil/contact cross-reference tables are derived.
- **Evidence:** the reference package names the target drawing ("SEE SAFETY GATE LOCK OVERVIEW
  DRAWING", "TO RIO1003") but not sheet and zone. **Ruled (T5):** keep the drawing reference and
  add sheet and zone, using generated drawing IDs: `CR110  11501/1.C4` [CV5].

### DP7 — A circuit repeated many times is drawn once as a typical

- **Kind:** default (proposed from the reference package; to confirm) · **Origin:** example, Oct 12 2026
- **Statement:** when N circuits differ only in tags and numbers, draw one page with placeholders
  (`[W] [X] [Y] [Z]`) and a cross-reference table listing each instance's values; the page title
  uses a wildcard (`RIO120#`).
- **Model:** each instance is still a full set of model objects (BOM, wire lists and the web viewer
  see every instance). The typical is a presentation choice, not a shortcut in the model [M3].
- **Examples:** [reference package](../examples/reference-package-notes.md): eight identical CAN interface circuits on one page.

### DP8 — Small field I/O modules may share a page

- **Kind:** default (proposed) · **Origin:** example, Oct 12 2026
- **Statement:** consecutive IP67 field modules on one chain segment can be drawn on one pictorial
  page, titled with the range. Cabinet I/O cards keep one page each [DP3].
- **Examples:** [reference package](../examples/reference-package-notes.md): pages titled `FIO2003-FIO2005` and `FIO2101-FIO2108`.

### DP9 — Cabinet boundaries are drawn where a circuit crosses them

- **Kind:** default (proposed) · **Origin:** example, Oct 12 2026
- **Statement:** when a circuit on one page runs into another cabinet, a dashed boundary line is
  drawn and labelled with both cabinet names (`CAB1 │ CAB2`). The crossing must correspond to a
  declared interconnect in the model [I7].
- **Examples:** [reference package](../examples/reference-package-notes.md): an output wired to a relay in the next cabinet; a power page boundary.

### DP10 — Device configuration is shown with the device

- **Kind:** default (proposed) · **Origin:** example, Oct 12 2026
- **Statement:** settings a technician must make on the device (DIP-switch safety address, jumpers)
  are drawn on the device's I/O page, from model parameters.
- **Examples:** [reference package](../examples/reference-package-notes.md): TwinSAFE address switches drawn under the module.

### DP11 — OEM installation excerpts may be placed on a device page

- **Kind:** default (proposed) · **Origin:** example, Oct 12 2026
- **Statement:** a device page can carry figures or tables from the OEM manual (wiring steps, torque
  table), each with its source document, revision and page, and only where its use is permitted
  (see `product.md` §5 on document rights).
- **Examples:** [reference package](../examples/reference-package-notes.md): a VFD page with installation steps and a torque table.

### DP12 — Package order follows the fieldbus topology

- **Kind:** default · **Origin:** example, confirmed by the user Oct 12 2026 (ADR 0010)
- **Statement:** cabinets appear in the order they occur on the fieldbus; within a cabinet, I/O
  pages follow slot order; a safety function page follows the last module it uses; field I/O
  chains are drawn with the cabinet they start from.
- **Examples:** [reference package](../examples/reference-package-notes.md): the drawing list.

### Tensions and their rulings

| ID | Between | Situation | Default until ruled |
|---|---|---|---|
| T1 | DP3 vs DP4 | A safety-related signal wired to an I/O channel: I/O page or safety page? | **Answered by the reference package**: wiring on the safety page; the I/O page keeps the channel with a "see overview" note |
| T2 | DP1 vs DP3 | A field device with an isometric view: its own device page too? | **Answered** by the reference package: no; field devices are pictorial on their field I/O page. Device pages (PS-1) are for complex devices such as a VFD |
| T3 | DP2 vs page density | A device with too many connections for one 11 × 17 page | Continue on a second page with the same anchor and DP6 cross-references |
| T4 | CV3 vs the full content list in `product.md` §6 | Where architecture views, safety pages, device pages, network topology, terminal plans and cable schedules go | **Resolved** (ADR 0010): network overview in front matter; safety pages inline (DP12); device overviews after the cabinets; terminal plans and cable schedules in the workbook and web viewer, not the drawing package |
| T5 | DP6 vs reference practice | Sheet and zone on every off-page reference? | **Resolved** (ADR 0010): yes, with the drawing ID: `CR110  11501/1.C4` |

## 4 Symbols and device views

- Schematic symbols are purpose-built with verified pins; a STEP projection never supplies
  electrical meaning [D5].
- Multi-unit devices (a contactor's coil, main contacts and auxiliaries) are split by symbol unit
  across pages as the page strategy dictates; the device is still one object and one BOM line.
- Isometric views come from the STEP pipeline ([`product.md` §5](product.md#5-components-circuits-and-assets)),
  scaled to a consistent size per page.
- Schematic I/O modules are drawn as a box with the part number and short description above
  (`EL1859` / `8DI/8DO`), channel names inside, terminal numbers in small boxes at the edge, signal
  descriptions outside on the left and field devices on the right; unused channels are drawn.

## 5 Sheet format

- **Size:** 11 × 17 in, landscape (17 in wide × 11 in high), for every drawing-package sheet. PDF
  page size and DXF paper space match the physical print size [D1].
- **Title block:** bottom right, inside the printable border, reserved by the layout engine so
  nothing overlaps it. Contents: project/machine, drawing title, drawing ID [D8], sheet n of N,
  revision, date, drawn/checked/approved (approval fields only from recorded approvals), scale or
  "not to scale". Exact dimensions and styling are taken from the user's reference package.
- **Legibility:** text and line weights checked at 100 % print size; panel-view scales stated;
  details enlarged onto extra sheets rather than shrunk below legible size.
- **Zones:** 8 columns (1–8) × 6 rows (A–F), marked on all four sides.
- **Measured style** from the reference DXF (detail in the [reference package](../examples/reference-package-notes.md) §4; values
  in `profiles/author/conventions.yaml` → `drawing_style`): ISOCPEUR font; 2.0 mm general text,
  1.5 mm small annotation, 5.0 mm large labels; I/O channel pitch 12.13 mm; title block about
  127 × 43 mm; dimensions in mm with inches in brackets; scale "N.T.S".
- **DXF structure** (observed): every sheet drawn in model space at 1:1 mm with one paper-space
  layout per sheet named after the drawing. LCAD's DXF export should follow the same structure so
  the files feel familiar in AutoCAD LT (to confirm).

*Print acceptance:* every PDF page is 17 × 11 in landscape with the bottom-right title block and no
content outside the border or in the reserved block; a dense schematic, an architecture sheet and
a dimensioned panel sheet are checked at 100 %.

## 6 Open questions for the user

Answered by the reference package: T1, T2, most of T4, DP1 scope, I/O page order, RIO/FIO naming,
sheet numbering. Answered by the user: CV2 wire labels (ADR 0009); CV3 package order, T5 and drawing
IDs (ADR 0010). Still open:

1. **DP7–DP11:** confirm the proposed entries.
2. **DP3:** the reason power goes on separate pages (for the "why" line).
3. **DP4:** where a safety function stops fitting one page.
4. **T3:** a device with too many connections for one page.
5. **ORG** wire colour: what it signifies.
6. Device tag numbers: what the tens digit groups (10s vs 20s).
