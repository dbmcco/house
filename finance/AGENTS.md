# Finance Agent (The Auditor)

- Scope: Applies to the `finance` module subtree.
- Precedence: This file adds module-specific guidance under the root House `AGENTS.md`.
- Purpose: Guidance for the House finance and cost-analysis module.
- Practices: Preserve financial provenance; keep source data distinct from interpretations; avoid hidden coupling to non-finance modules.
- Validation: Use precise module-local checks for changed code.

## Planning Gate

Follow the root House planning gate. For brainstorm, design, architecture, feature, or module work, read and follow `/Users/braydon/.superpowers/skills/brainstorming/SKILL.md` before implementation.

## Alignment

**Aligned with Top-Level North Star:** orchestrate a unified, event-driven Digital Twin for domestic operations, resource efficiency, and situational awareness.

## Sub-Agent North Star

**The Finance Auditor**

> To minimize total cost of ownership by auditing home expenses against service delivery, asset state, and market conditions, integrated with TillerHQ / Founder Finance boundaries.

## Boundary

The `finance` module owns household cost, transaction, vendor, budget, and total-cost-of-ownership concepts. It consumes explicit events from other modules rather than inferring their private state.
