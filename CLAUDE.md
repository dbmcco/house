@../../claude-workspace/memories/base/interaction-style.md
@../../claude-workspace/memories/base/core-principles.md
@../../claude-workspace/memories/base/code-standards.md
@../../claude-workspace/memories/base/version-control.md
@../../claude-workspace/memories/workflows/tdd.md
@../../claude-workspace/memories/workflows/llm-driven-development.md
@../../claude-workspace/memories/workflows/rifts.md

# House Digital Twin

Project context for agents working on the House ecosystem.

## Purpose

Build a modular home Digital Twin that coordinates energy, property, finance, and core orchestration signals to improve household efficiency, cost, safety, and situational awareness.

## Current Phase

Early bootstrap and design. Prefer discussion, brainstorming, and clear module contracts before implementation.

## Modules

- `core`: lightweight event routing, module registry, and shared contracts.
- `energy`: solar, battery, demand, weather, and tariff-aware optimization.
- `finance`: home expense auditing and total cost of ownership analysis.
- `property`: maintenance, physical assets, sensors, security, and environmental state.

## Working Rules

- Use the brainstorming superpower before new feature/module work.
- Keep module state and logic isolated unless a shared boundary is explicitly designed.
- Treat the event bus as transient coordination, not historical storage.
- Prefer explicit data contracts and provenance over informal string matching or hidden coupling.
- Do not optimize one module at the expense of the root House North Star.

## Tech Notes

- Runtime may vary by module: Node.js, Python, or shell as appropriate.
- Persistence should be module-local until a shared store is designed.
- Finance integrations should align with TillerHQ / Founder Finance boundaries.
