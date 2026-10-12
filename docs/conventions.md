---
purpose: Per-project conventions — naming, abbreviations, wire labels, numbering and package structure — and the profiles they are copied from
read-when: Creating a project; touching tags, wire labels, numbering, abbreviations, sheet titles or package/section structure; when a request contradicts a project's conventions
status: draft
---

# Conventions and profiles

Some choices are not right or wrong; they are how a particular designer, shop or customer does
things: what remote I/O is called, what a wire label looks like, what order the drawing package is
in. LCAD keeps these choices out of its code and out of the model. They live in a **conventions
file** that each project owns [C1, C3].

## 1 Profiles and projects

- A **profile** is a named, reusable set of conventions and a sourcing policy, kept in the LCAD
  repository under [`profiles/`](../profiles/) (or in an organisation's own location).
  [`profiles/author/`](../profiles/author/) holds the LCAD author's conventions.
- **Creating a project copies a profile** into the project's `profile/` folder and records the
  source profile and its hash in `profile/origin.yaml` [C1]. From then on the project's copy is
  authoritative. Another project can start from another profile, or from the same one and diverge.
- **Profiles do not change existing projects.** Editing a profile later affects only projects
  created afterwards. LCAD can show the differences between a project's conventions and its source
  profile and offer to pull specific changes in, as a reviewable change [C1].
- **Changing a project's conventions** goes through the `change_convention` operation, which shows
  the impact first (e.g. how many released wire labels would change) [C5].

```text
machine-project/profile/
  conventions.yaml     # naming, abbreviations, wire labels, numbering, package structure
  sourcing.yaml        # sourcing policy (rules B1, SP*)
  origin.yaml          # copied from: profiles/author @ <hash>, date
```

## 2 What conventions control, and what they cannot

| Conventions control | Conventions cannot change |
|---|---|
| Abbreviations and class labels (e.g. RIO/FIO/CIO) | Any **hard** rule in [`RULES.md`](../RULES.md) |
| Tag scheme | Integrity rules (I*), model rules (M*) |
| Wire label format and numbering scheme | Sheet size and title block position (D1) |
| Terminal and cable naming | Cross-reference requirement (DP6) |
| Package structure: sections, their order and what goes in each | Release requirements (W4) |
| Sheet numbering and title formats | |
| Project-level choices among **default** drawing-philosophy entries (e.g. turning DP1 off) | |

A convention that would override a hard rule is rejected. A convention that departs from a
*default* rule is a scoped exception, and it is recorded in the conventions file with its reason
[C2].

The engine reads every convention from the project's file; no abbreviation, label format or
section order is hard-coded [C3].

## 3 Conflicts with conventions

Inside a project, a request that contradicts that project's conventions (for example "label this
wire K12-A1" in a project whose labels are numeric only) is handled like any other conflict [C4]:
the request is not applied silently; LCAD shows the convention, explains the conflict, and offers
(a) follow the convention, (b) change the convention for this project, or (c) a recorded exception
for that object. During LCAD's own development the same protocol applies to the author profile,
whose values are listed in `RULES.md` (CV*).

## 4 Conventions file reference

The schema below is a draft; it will be adjusted against the user's example package in Phase 0.

### I/O classes

Labels for classes of I/O. The **class of a module is derived from the model**, not typed in, so it
cannot drift [M3]: the conventions file gives each class a label and a derivation rule.

```yaml
io_classes:
  - label: CIO
    meaning: Chassis I/O, attached to the CPU
    derive: { topology_parent: cpu }
  - label: RIO
    meaning: Remote I/O modules or cards on a remote coupler
    derive: { topology_parent: coupler, ip_rating_below: IP67 }
  - label: FIO
    meaning: Field I/O, IP67, mounted on the machine
    derive: { ip_rating_at_least: IP67 }
```

Illustration with Beckhoff hardware: EL terminals directly on a CX controller would be CIO; EL
terminals behind an EK1100 coupler in a remote box would be RIO; EP/EPP box modules on the machine
would be FIO. If a module's tag or section disagrees with its derived class, that is a finding.

### Wire labels and numbering

Author profile (CV2, [ADR 0009](decisions/0009-numeric-wire-labels.md)): six digits `AAGGSS`, one
label per potential.

```yaml
wire_labels:
  format: "{area:02}{group:02}{seq:02}"
  one_label_per: potential
  io:      { area: module_station, group: module_slot, seq: terminal_index }   # RIO1001 t7 -> 100107
  circuit: { area: cabinet_area, groups: [90, 99], seq: drawing_order }         # area 10 power -> 109001
```

A potential tied to exactly one I/O terminal and no supply takes the I/O form, so its first four
digits are the module number. Every other potential takes a circuit-group label in its source's
area. Allocation and locking follow [`model.md` §7](model.md#7-numbering). Because labels encode the
area, moving a device into a cabinet with a different area number raises finding F6 in
[`integrity.md`](integrity.md).

### Package structure

The ordered sections of the drawing package. Every generated page belongs to exactly one section;
anything not claimed by an earlier section lands in the catch-all section [C6]. Author profile (CV3,
[ADR 0010](decisions/0010-package-structure-cross-references-drawing-ids.md)):

```yaml
package:
  sections:
    - { id: front, area: "00", contains: [cover, drawing_list, general_notes, control_system_overview] }
    - id: per_cabinet
      repeat_for: cabinet
      order: topology                       # cabinets in fieldbus order
      sections: [exterior, interior, assemblies, power, io]   # io: slot order, safety pages after their
                                            # last module, typicals, field I/O with its upstream cabinet
    - { id: device_overviews, contains: [{ page_strategy: PS-1 }] }
    - { id: misc, catch_all: true }
  not_in_drawing_package: [terminal_plans, wire_lists, cable_schedules, bom]
```

The engine reports what landed in the catch-all section so that "misc" stays a deliberate choice
rather than a place things disappear to.

### Drawing IDs and sheets

Author profile (CV4): five digits `AATNN`, generated, unique in the project and locked once
released [D8, M10].

| Part | Meaning |
|---|---|
| `AA` | `00` front matter; otherwise the cabinet's area number (same as circuit wire labels) |
| `T` | `0` cover/overviews · `1` exterior · `2` interior · `3` assemblies · `4` power · `5` I/O, safety, typicals, field I/O · `6` device overview · `9` misc |
| `NN` | sequence within area and type, in package order |

Examples: cover `00001`, cabinet area 10 exterior `10101`, its first I/O page `10501`. A drawing
can have several sheets (`1 OF 2`). Outside the sheet the ID carries the project prefix
(`<project>-10501`).

### Off-page references

Author profile (CV5): target tag plus drawing, sheet and zone, e.g. `CR110  11501/1.C4`; a
note-style reference may add the drawing title (`SEE 10502/1.B3  GATE LOCK OVERVIEW`) [DP6].

## 5 Where content goes (author profile)

| Content | Section |
|---|---|
| Cover, drawing list, general notes, control system (network) overview | front matter |
| Exterior and interior layouts, harness/assembly pages, power distribution | the cabinet's section |
| I/O pages, safety function pages, typicals, field I/O | the cabinet's I/O section |
| Device overviews (e.g. a VFD) | device overviews |
| Terminal plans, wire lists, cable and core schedules, cable entry schedule | not in the drawing package: workbook and web viewer |
| BOM | separate document |
| Legends, symbol key, anything unassigned | misc (reported) |
