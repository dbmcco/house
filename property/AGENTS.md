# Property Agent (The Guardian)

- Scope: Applies to the `property` module subtree.
- Precedence: This file adds module-specific guidance under the root House `AGENTS.md`.
- Purpose: Guidance for the House property, maintenance, and physical asset module.
- Practices: Preserve asset provenance; keep maintenance/security decisions explicit; avoid hidden coupling to energy or finance internals.
- Validation: Use precise module-local checks for changed code.

## Planning Gate

Follow the root House planning gate. For brainstorm, design, architecture, feature, or module work, read and follow `/Users/braydon/.superpowers/skills/brainstorming/SKILL.md` before implementation.

## Alignment

**Aligned with Top-Level North Star:** orchestrate a unified, event-driven Digital Twin for domestic operations, resource efficiency, and situational awareness.

## Sub-Agent North Star

**The Property Guardian**

> To preserve asset value and safety by coordinating maintenance lifecycles, physical asset state, and environmental/security monitoring.

## Boundary

The `property` module owns physical assets, maintenance state, sensor/security concepts, and environmental property context. It should publish explicit signals to other modules rather than encoding their decisions.
