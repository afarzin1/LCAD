---
purpose: ADR — keep object relations consistent through derived data, typed operations and global invariants, with the AI resolving findings rather than walking checklists; add the drawn/placed/procured BOM cross-reference
read-when: Implementing model changes, validators or BOM generation; deciding what the AI should do during a change
status: stable
---

# ADR 0007 — Deterministic change integrity and BOM cross-reference

**Status:** Accepted 2026-10-12.
**Context source:** user request, Oct 12 2026: moving a device to another cabinet must carry its
circuits, drawings and BOM location with it; prefer deterministic methods over an LLM working
through a checklist; everything on the BOM must be integrated and everything integrated must be on
the BOM.

## Options considered

1. **LLM checklist:** the AI searches for every occurrence and updates it. Flexible, but token cost
   grows with project size and a missed item goes unnoticed.
2. **Per-operation propagation code:** each operation lists its own side effects. Deterministic,
   but the lists drift and each new operation repeats the work.
3. **Derived data + typed operations + global invariants, AI for decisions only** (chosen).

## Decision

1. The model stores each fact once and derives the rest [M3], so most occurrences need no update.
2. Changes go through typed operations [I1] that apply single-answer consequences and preview an
   impact report [I2, I4].
3. A global invariant set runs after every change regardless of cause [I3]; operations do not own
   their checks.
4. Multi-answer consequences become findings with options. The AI recommends options; the user
   confirms where approvals are affected [A4].
5. A drawn/placed/procured expected-presence matrix per object class, plus BOM-line traceability,
   implements the BOM cross-reference as a release gate [I5, I6].

Detail: [`integrity.md`](../integrity.md).

## Consequences

- Token use scales with the size of the decision, not the project.
- A missed consequence cannot be silent: it is an invariant failure and blocks release.
- Requires the model to be designed for derivation from the start (locations as a tree, all
  relations by ID, interconnects as explicit cables) — this is why multiple cabinets are in schema
  v0 [M9].
- The invariant list must be kept complete; a gap in invariants is a gap in integrity. New object
  types must come with their invariants.
