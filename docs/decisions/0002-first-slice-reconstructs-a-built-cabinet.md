---
purpose: ADR — the first milestone is regenerating a cabinet that was already built, from a hand-written model; AI intake and architecture come later
read-when: Planning phases, or tempted to start with the AI intake or the architecture diagram
status: stable
---

# ADR 0002 — The first slice reconstructs an already-built cabinet

**Status:** Accepted 2026-10-12.

## Context

The original Phase 3 "first vertical slice" included architecture review, a 3D concept, schematics,
reconciled 3D and dimensioned layouts, the offline web application with tracing, the print PDF,
cable entry, conductor estimates and the TwinCAT adapter, which is most of the product. The parts
that are new and risky are the model and the deterministic renderer (especially schematic page
composition). AI intake and architecture diagrams are the least distinctive and the easiest to add
later, and the hardest to trust without a model underneath them.

## Decision

1. **Phase 0** writes one real, already-built and documented cabinet in the schema by hand. The
   schema is revised until that is natural ([`model.md`](../model.md)).
2. **Phase 1** generates from that model: I/O module and control-power sheets, BOM, I/O list, wire
   list and terminal plan, and compares them with the real package. Typed operations, invariants and
   the BOM cross-reference are built here, because they are what make generation trustworthy.
3. AI intake, requirements and architecture review move to Phase 3, after the model and renderer
   are proven.
4. "Can LCAD reproduce a cabinet you already built?" is the acceptance test for the core.

## Consequences

- The schema is shaped by a real cabinet rather than by guesses.
- There is a ground truth for every generated output.
- The user workflow in [`product.md` §2](../product.md#2-user-workflow-and-project-states) is unchanged
  for end users; only the build order changes.
- A cost: the AI-assisted workflow, which is the most visible feature, comes later.
