---
purpose: ADR — split the monolithic spec into a router, a rules registry and focused documents; adopt a conflict protocol for contradictory requests
read-when: Wondering why the docs are organised this way, or why an agent stopped to ask about a conflict
status: stable
---

# ADR 0001 — Rules registry, document router and conflict protocol

**Status:** Accepted 2026-10-12.
**Context source:** review of `archive/lcad-expanded-roadmap-2026-10-11.md`; user request of
2026-10-12.

## Context

The October 11 specification was one 70 KB document that mixed product intent, shop policy and
implementation detail, and stated several rules three or four times (the sourcing rules appeared in
§6, §10, Phase 2 and §16). It was hard for a contributor or an agent to hold in context, and a rule
restated in several places drifts. The user also expects to contradict earlier requirements during
development and wants contradictions caught, explained and confirmed rather than resolved silently.

## Decision

1. `AGENTS.md` is the single agent entry point and routes each topic to one owning document [P1].
2. `RULES.md` lists every normative rule as one line with an ID, kind, owner, origin and check
   [P2]. Detail lives only in the owner.
3. **Conflict protocol** [P3]: a request that contradicts or strains a rule stops the affected work;
   the agent cites the rule IDs and their origin, explains the conflict and consequences, offers
   options (keep, change, scoped exception), recommends one, and waits for explicit confirmation.
   Conflicts between two existing rules are reported the same way.
4. Confirmed changes update `RULES.md`, the owner and, when a decision is reversed, an ADR, in one
   change [P4].
5. The original spec is archived unchanged as history.

## Consequences

- An agent can check a request against ~70 one-line rules instead of re-reading the full spec.
- The `origin` column lets an agent say "you asked for X on Oct 11; this contradicts it" instead of
  presenting a rule as arbitrary.
- The `check` column shows which rules depend on review alone, which identifies the next validators
  to write (the approach LHMI uses for its conformance gates).
- Cost: every rule change touches two files. Accepted, because the registry is what makes conflict
  detection cheap.
