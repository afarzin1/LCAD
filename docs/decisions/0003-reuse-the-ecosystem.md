---
purpose: ADR — build the TwinCAT handoff on tcDeploy, host offline docs on LHMI, and link devices to LPLC block types instead of building parallel tools
read-when: Touching the TwinCAT export, the offline documentation host, or PLC/HMI integration
status: stable (hosting choice for docs open until Phase 0)
---

# ADR 0003 — Reuse tcDeploy, LHMI and LPLC

**Status:** Accepted 2026-10-12.

## Context

The original spec planned a new Windows TwinCAT XAE adapter and a new offline HMI hosting profile,
and listed PLC/HMI handoff as a late (P2) feature. The same author already maintains:

- **tcDeploy**, which drives the XAE Shell over COM on a Windows build box from Linux, with a job
  queue, structured errors and an approval gate before activation.
- **LHMI**, an LLM-first web HMI whose Python backend serves a static build on the machine, with an
  apps mechanism for opt-in sidecar services and its own target hardware experience.
- **LPLC**, a TwinCAT library whose device blocks each expose one `.HMI` struct that LHMI binds to.

## Decision

1. **TwinCAT I/O** [X1]: LCAD's output is the documented, vendor-neutral I/O package. Applying it to a
   TwinCAT project is a tcDeploy command that runs on the existing build box through the Automation
   Interface (verify, dry-run diff, apply to a copy, read back, report). Export never activates a
   configuration [X2].
2. **LPLC** [X3]: a device may name an LPLC block type. LCAD exports PLC instance declarations,
   seeding each block's device name from the LCAD tag and its I/O links from the channel map. LHMI
   continues to generate bindings from its own PLC scan (LHMI rule R4); LCAD never writes LHMI
   bindings.
3. **LHMI** [X4]: the machine's LHMI backend is the candidate host for the offline documentation
   bundle (static route or LHMI app), sharing LHMI's target hardware profile. Decided in Phase 0.
4. PLC/HMI handoff moves from P2 to P1.

## Consequences

- No second COM automation layer to maintain; the Windows-side quirks already solved in tcDeploy
  (sessions, wedged shells, line endings) are reused.
- One model drives the cabinet, the PLC device instances and, through the PLC, the HMI faceplates.
  No commercial tool covers that span.
- LCAD's export contract must stay usable without tcDeploy, LPLC or LHMI, so other tools can consume
  it.
- tcDeploy gains a feature that must respect its own safety rules (never deploys TwinSAFE).
