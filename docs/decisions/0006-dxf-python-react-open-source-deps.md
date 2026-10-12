---
purpose: ADR — DXF instead of DWG; Python engine; React viewer; dependencies must permit open-source distribution
read-when: Choosing a library, adding a CAD format, or starting implementation
status: stable
---

# ADR 0006 — DXF only, Python engine, React viewer, open-source-compatible dependencies

**Status:** Accepted 2026-10-12. Resolves the original spec's open decision on "mandatory DWG
fidelity".

## Context

- Reliable DWG writing effectively requires the Open Design Alliance's commercial libraries; the
  open alternative (LibreDWG) is GPL and incomplete. Either would constrain an open-source LCAD.
  DXF is open, widely imported, and sufficient for drawing exchange.
- The original spec left the engine's language undecided. LHMI's backend and tooling are Python;
  LHMI's frontend is React.
- Open CASCADE (LGPL) is the candidate geometry kernel for STEP processing.

## Decision

1. CAD export is DXF with paper-space layouts matching the 11 × 17 sheets; DWG is not supported [D6].
2. The engine (model, validators, operations, BOM, schedules, sheet composition, SVG/PDF, DXF) and
   the geometry worker are Python [X6].
3. The viewer and authoring UI are React; all project files and exports are framework-independent
   JSON/YAML with JSON Schema [X6].
4. Every dependency must allow open-source distribution of LCAD [X7]; licences are checked when a
   dependency is added.

## Consequences

- Users who need DWG convert DXF in their own CAD tool.
- Python shares tooling, testing and packaging habits with the LHMI backend.
- The Open CASCADE Python bindings and their packaging are a Phase 0 proof item.
