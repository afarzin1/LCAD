# AGENTS.md — LCAD router

This file is the single entry point for **every** AI agent working on LCAD (Claude Code, Codex,
Cursor, Copilot, Gemini CLI, …). There is deliberately no tool-specific instruction file, so there
is one set of instructions to keep current. It routes you to the document that owns each topic and
defines the **conflict protocol** you must follow when a request disagrees with an existing rule.

**LCAD** turns machine requirements into a reviewed electrical design and a coordinated cabinet
package: schematics, panel layouts, BOM, offline web documentation and a TwinCAT I/O handoff, all
generated from one versioned engineering model. It is the electrical-design companion to
[LHMI](https://github.com/afarzin1/LHMI) (web HMI), [LPLC](https://github.com/afarzin1/LPLC)
(TwinCAT PLC library) and [tcDeploy](https://github.com/afarzin1/tcDeploy) (TwinCAT build/deploy from Linux).

> **Status: design stage.** Nothing described in `docs/` is implemented unless
> [`ROADMAP.md`](ROADMAP.md) says so. Treat the documents as intent, not as a description of code.

## Read before doing anything

1. **[`RULES.md`](RULES.md), completely.** Every normative rule has an ID and one owning document.
   It is the index you check a request against. It is short on purpose.
2. **The owning document(s) for the area you are touching** (router below).
3. **[`ROADMAP.md`](ROADMAP.md)** for what is in scope now. Update it in the same change when you
   finish or re-plan work.

## Router

| If the task touches… | Read | Rule IDs |
|---|---|---|
| What LCAD is, user workflow, project states, outputs | [`docs/product.md`](docs/product.md) | W, S, X |
| The engineering model, file layout, IDs, units, project repos | [`docs/model.md`](docs/model.md) | M |
| Changing the model: moving, renaming, replacing, deleting objects; invariants; BOM cross-reference | [`docs/integrity.md`](docs/integrity.md) | I |
| How drawings are composed: page strategies, iso views, what goes on which page | [`docs/drawing-philosophy.md`](docs/drawing-philosophy.md) | D, DP |
| Naming, abbreviations, wire labels, numbering, package sections, profiles | [`docs/conventions.md`](docs/conventions.md) + [`profiles/`](profiles/) | C, CV |
| Part selection, suppliers, price, lead time, quotes | [`profiles/author/sourcing.yaml`](profiles/author/sourcing.yaml) + [`docs/product.md` §7](docs/product.md#7-bom-and-procurement) | B, SP |
| TwinCAT, LPLC, LHMI, tcDeploy integration | [`docs/product.md` §9](docs/product.md#9-ecosystem-integration) + [ADR 0003](docs/decisions/0003-reuse-the-ecosystem.md) | X |
| Why something was decided | [`docs/decisions/`](docs/decisions/) | — |
| Phases, scope, open questions | [`ROADMAP.md`](ROADMAP.md) | — |
| The pre-restructure spec (history only, not normative) | [`archive/`](archive/) | — |

## Conflict protocol (rule P3) — mandatory

LCAD has many requirements and they will sometimes contradict each other, or a new request will
contradict an old one. **Never resolve a contradiction silently**, in either direction: do not
quietly ignore the new request, and do not quietly drop the old rule.

**Before acting on any request that changes behavior, design, a document, or a convention:**

1. **Check it against [`RULES.md`](RULES.md).** Search by area and by keyword. Also check the
   owning document's detail and any ADR that the rule cites.
2. **Classify what you find:**
   - **Consistent** — proceed.
   - **Refinement** — adds detail without changing an existing rule's meaning. Proceed, add or
     update the rule, and mention it in your reply.
   - **Tension** — both can hold, but one weakens the other (e.g. "keep each safety function on one
     page" vs. "one page per I/O module" when a safety input is on a standard I/O card). Proceed
     only after stating the trade-off and which rule wins by default.
   - **Contradiction** — both cannot hold. **Stop work on the affected part.**
3. **For a tension or contradiction, ask the user, in this shape:**

   > **Conflict with [ID] (owner: `docs/…`)**
   > - **Existing rule:** quote it, with its origin (who/when/ADR).
   > - **Your request:** quote or paraphrase it.
   > - **Why they conflict:** one or two sentences, concrete.
   > - **Consequences:** what each choice changes downstream (other rules, outputs, phases).
   > - **Options:** (a) keep the rule, (b) change the rule for everyone, (c) scoped exception
   >   (this project / this sheet / this device class), (d) something else you propose.
   > - **My recommendation** and why.

4. **Wait for an explicit answer.** Silence or a vague reply is not confirmation; ask again or work
   on unaffected parts. Work that does not depend on the conflict may continue.
5. **Record the outcome in the same change** (rule P4): update the `RULES.md` row, the owning
   document, and add or amend an ADR in `docs/decisions/` when a decision is reversed or reshaped.
   A scoped exception is recorded where it applies (a machine project's `profile/conventions.yaml`
   for a convention, its `reviews/decisions.yaml` otherwise; the owning doc for LCAD itself) and
   referenced from the rule.

Also apply the protocol when **two existing rules** turn out to conflict, even if nobody asked for
anything. Report it; do not pick a winner yourself.

The same protocol exists inside the product: when a machine project's requirements contradict each
other or contradict LCAD's defaults, LCAD raises a finding and asks (rule W7). Development-time and
design-time conflicts are handled the same way.

## Working rules

- **Rules live in one place.** A rule's normative wording is in `RULES.md` plus exactly one owning
  document. Other documents link to it rather than restating it (rule P2). If you find a rule
  stated normatively in two places, report it and consolidate.
- **Front matter.** Every Markdown file under `docs/` starts with `purpose`, `read-when`, `status`
  front matter, as in LHMI.
- **Decisions are ADRs.** `docs/decisions/NNNN-short-name.md`, numbered, never renumbered;
  superseded ADRs stay and say what superseded them.
- **No site data.** Never commit real customer or machine names, hostnames, IP addresses, quotes,
  prices or credentials to this repository. Use neutral placeholders (rule P7).
- **Reference documents from real projects** (drawing packages, BOMs, DXF) go in
  `examples/private/`, which is git-ignored and never published (rule P8). Extract what defines the
  standard into the profile, `docs/drawing-philosophy.md` and `examples/reference-package-notes.md`,
  without customer, company or project names, product-specific function names, or page references
  to the private files.
