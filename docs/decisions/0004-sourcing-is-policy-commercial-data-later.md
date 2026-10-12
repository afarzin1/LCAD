---
purpose: ADR — supplier and manufacturer rules become a policy file instead of product requirements; live price/lead-time lookups move to a later phase
read-when: Touching part selection, suppliers, pricing, lead times or quotes
status: stable
---

# ADR 0004 — Sourcing is policy; commercial data comes later

**Status:** Accepted 2026-10-12.

## Context

The original spec built one shop's purchasing rules into the product: McMaster-Carr as default,
DigiKey/Mouser as alternates, Saginaw/Rittal enclosures only, Radwell pricing and user lead times
for Beckhoff, user quotes for non-online parts, Icotek cable entry. These are valid rules for that
shop, but LCAD is intended to be open source alongside LHMI and LPLC (MIT). The spec also required
per-line price and lead time from Phase 2.

Feasibility check (Oct 12 2026): DigiKey and Mouser have public APIs. McMaster-Carr's Product
Information API is limited to approved customers, needs a client certificate and per-product
subscriptions, and documents price but not lead time. No public Radwell API is known.

## Decision

1. The rules become a sourcing policy file, evaluated deterministically [B1]. (ADR 0008 later placed
   it in profiles: `profiles/author/sourcing.yaml`, copied to each project's `profile/sourcing.yaml`.) The author's rules are
   the shipped example and remain in force for the author's projects (SP1–SP6 in `RULES.md`).
2. Price and lead-time fields and statuses exist from the first BOM [B2], but automated lookups
   are Phase 5 and never block engineering [B6].
3. Until lookups exist, and wherever an API is unavailable, prices and lead times are user-entered
   with source and date.

## Consequences

- Another organisation can use LCAD by replacing one file.
- The shop's rules are still enforced and still covered by the conflict protocol, because they are
  in `RULES.md`.
- Phase 2 shrinks considerably; the engineering core is not held back by supplier integrations.
