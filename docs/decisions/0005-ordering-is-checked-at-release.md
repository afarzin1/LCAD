---
purpose: ADR — replace the hard generation gates (architecture approval, mandatory 3D concept) with draft stamps and a release check; allow a 2D/table space estimate
read-when: Touching project states, approvals, the space estimate, or anything that blocks generation
status: stable
---

# ADR 0005 — Workflow order is checked at release, not forced at generation

**Status:** Accepted 2026-10-12. **Supersedes** the original spec's §2 statement that "the generation
service must enforce this order", the hard block on draft schematics before architecture approval,
and the requirement that the pre-schematic concept be 3D.

## Context

The original spec blocked all schematic generation, including drafts, until the architecture was
approved, and blocked schematics until a 3D interior/exterior concept was issued.

- Projects are Git repositories that users and agents may edit directly, so a generation-time block
  cannot really be enforced; it only gets in the way of the LCAD tool itself.
- A 3D concept before schematics needs component envelopes that do not exist yet; for small panels
  and retrofits it is effort with little value.
- What the order actually protects is the **released** package.

## Decision

1. Schematics generated before a current architecture approval, or before a space estimate for the
   approved revision, are allowed but stamped **PRE-APPROVAL DRAFT** and raise a warning [W2].
2. The pre-schematic space estimate can be a table, a 2D arrangement or a 3D concept [W3].
3. Release requires current architecture approval, a space estimate for that revision, layout
   reconciled with schematics, no blocking findings and a recorded engineering approval [W4].
4. Approval binding to revision hashes and stale-marking of dependent approvals are unchanged [W6].

## Consequences

- The intent of the original ordering is kept where it matters (nothing is released out of order)
  without fighting users during design.
- Draft stamps make it obvious when a sheet was produced before its prerequisites.
- If a user later wants the hard block back for a particular organisation, it can be a policy flag;
  that would go through the conflict protocol as a change to W2.
