# LCAD

**LCAD** turns machine requirements into a reviewed electrical design and a coordinated cabinet
package (schematics, panel layouts, BOM, wire and terminal schedules, offline web documentation,
print PDF and a TwinCAT I/O handoff), all generated from one versioned engineering model.

AI proposes; a deterministic engine validates, propagates changes and builds every output; an
engineer approves. A reviewed project rebuilds and is viewed without any LLM call.

> **Status: design stage.** Nothing is implemented yet. See [`ROADMAP.md`](ROADMAP.md).

LCAD is the electrical-design companion to:

- [**LHMI**](https://github.com/afarzin1/LHMI): LLM-first web HMI for industrial machines
- [**LPLC**](https://github.com/afarzin1/LPLC): TwinCAT 3 library of device and machine blocks
- [**tcDeploy**](https://github.com/afarzin1/tcDeploy): build and deploy TwinCAT 3 projects from Linux

## Documents

| Document | What it covers |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Entry point for AI agents: document router and the conflict protocol |
| [`RULES.md`](RULES.md) | Every rule, one line each, with ID, owner and origin |
| [`ROADMAP.md`](ROADMAP.md) | Build phases, status and open decisions |
| [`docs/product.md`](docs/product.md) | What LCAD does: workflow, intake, architecture, outputs, BOM, standards, integration |
| [`docs/model.md`](docs/model.md) | The engineering model and project repository layout (schema v0 draft) |
| [`docs/integrity.md`](docs/integrity.md) | How changes propagate: typed operations, invariants, BOM cross-reference |
| [`docs/drawing-philosophy.md`](docs/drawing-philosophy.md) | How drawings are composed: page strategies and design philosophy |
| [`docs/decisions/`](docs/decisions/) | Architecture decision records |
| [`docs/conventions.md`](docs/conventions.md) | Per-project conventions: naming, wire labels, numbering, package structure |
| [`profiles/`](profiles/) | Reusable convention and sourcing profiles copied into new projects |
| [`examples/`](examples/) | Reference drawings that back the drawing philosophy |
| [`archive/`](archive/) | The original Oct 11 2026 specification (history, not normative) |
