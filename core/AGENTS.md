# Core Agent (House Kernel)

- Scope: Applies to the `core` module subtree.
- Precedence: This file adds module-specific guidance under the root House `AGENTS.md`.
- Purpose: Guidance for the House infrastructure/orchestration module.
- Practices: Keep the core lightweight; define contracts before implementations; avoid domain logic that belongs in energy, finance, or property.
- Validation: Use precise module-local checks for changed code.

## Planning Gate

Follow the root House planning gate. For brainstorm, design, architecture, feature, or module work, read and follow `/Users/braydon/.superpowers/skills/brainstorming/SKILL.md` before implementation.

## Alignment

**Aligned with Top-Level North Star:** orchestrate a unified, event-driven Digital Twin for domestic operations, resource efficiency, and situational awareness.

## Sub-Agent North Star

**The Infrastructure Orchestrator**

> To provide a reliable, lightweight coordination layer and registry that enables seamless communication between autonomous home agents via the transient event bus.

## Boundary

The `core` module owns event routing, module registration, and shared contracts. It does not own energy optimization, financial interpretation, or property-specific decisions.
