---
purpose: ADR — conventions (naming, wire labels, numbering, package structure) are per-project files copied from reusable profiles; the sourcing policy moves into the same profile mechanism
read-when: Touching tags, labels, numbering, package sections or profiles; wondering why a convention is not in code
status: stable
---

# ADR 0008 — Per-project conventions copied from profiles

**Status:** Accepted 2026-10-12.
**Context source:** user request, Oct 12 2026: a per-project convention list, with the author's
own conventions (RIO/FIO/CIO, numeric wire labels, package order cover/TOC → exterior → interior →
power distribution → I/O wiring per card → misc) copied into a project, while another project may
use different ones.

## Conflict check (rule P3)

- **Refinement:** the tag scheme and numbering policy were in `lcad.project.yaml` (`model.md`);
  they move to the project's conventions file. No change in meaning.
- **Refinement:** numeric-only labels narrow the numbering options already allowed by M10.
- **Tension T4:** `product.md` §6 listed package content with no section structure, including
  architecture views, safety, terminals and cable schedules. The author's structure has no named
  section for these. Resolved by default: `product.md` lists *content types*; conventions define
  *sections*; unassigned content goes to the catch-all Misc section until the user rules. Recorded
  as T4 in `drawing-philosophy.md`.
- No hard rule is contradicted.

## Decision

1. Each project owns `profile/conventions.yaml`, copied from a profile at creation, with its origin
   recorded; profile edits never change existing projects [C1].
2. Conventions control naming, abbreviations, labels, numbering, package structure and choices
   among *default* rules; they cannot override hard rules [C2]. Nothing they control is hard-coded
   [C3].
3. Contradicting a project's conventions triggers the conflict protocol [C4]; changing one goes
   through `change_convention` with an impact report [C5].
4. The package is an ordered list of sections; every page is in exactly one; the catch-all's
   contents are reported [C6].
5. I/O class (CIO/RIO/FIO) is derived from topology, location and IP rating; the convention only
   supplies labels and derivation rules [M3].
6. Profiles live in `profiles/`. The author profile holds CV1–CV3 and the sourcing policy, which
   moves from `policy/sourcing.example.yaml` to `profiles/author/sourcing.yaml`.

## Consequences

- One mechanism for everything that is a preference rather than a rule: conventions and sourcing.
- The author's conventions are listed in `RULES.md` (CV*), so contradicting them during LCAD's
  development is caught like any other rule.
- Two new invariants (INV-18, INV-19) check section membership, I/O class and label format.
