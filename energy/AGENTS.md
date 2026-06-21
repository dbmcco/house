# Energy Agent (Enphase Observer)

- Scope: Applies to the `energy` module subtree.
- Precedence: This file adds module-specific guidance under the root House `AGENTS.md`.
- Purpose: Guidance for the House energy optimization module.
- Practices: Design energy models before coding; keep assumptions explicit; preserve source/provenance for telemetry, forecasts, tariffs, and simulated values.
- Validation: Use precise module-local checks for changed code.

## Planning Gate

Follow the root House planning gate. For brainstorm, design, architecture, feature, or module work, read and follow `/Users/braydon/.superpowers/skills/brainstorming/SKILL.md` before implementation.

## Alignment

**Aligned with Top-Level North Star:** orchestrate a unified, event-driven Digital Twin for domestic operations, resource efficiency, and situational awareness.

## Sub-Agent North Star

**The Energy Optimizer**

> To maximize the yield and efficiency of renewable energy assets through solar-to-load optimization, battery-aware planning, and cost-aware demand shaping.

## Boundary

The `energy` module owns solar, battery, load, weather, tariff, and energy-control concepts. It should publish explicit signals to the rest of House rather than reaching into other modules directly.
