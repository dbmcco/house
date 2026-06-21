# AGENTS.md

- Scope: Applies to this repository subtree.
- Precedence: Deeper `AGENTS.md` files override this file. System, developer, and user prompts take precedence.
- Purpose: Agent guidance for the House Digital Twin ecosystem.
- Practices: Keep edits minimal and scoped; preserve module boundaries; discuss and design before building new behavior.
- Validation: Documentation-only changes require clarity review. Code changes should use precise repo/module checks relevant to the changed module.

## Planning Gate

- For brainstorm, design, architecture, feature, or module work, read and follow `/Users/braydon/.superpowers/skills/brainstorming/SKILL.md` before implementation.
- Do not implement, scaffold, edit code, commit, or push feature work until the design direction is approved.
- If the user says “brainstorm,” “discuss,” “design,” “use superpowers,” or “we are not building,” stop implementation and return to design discussion.

## Application Boundary

This repo owns the House Digital Twin ecosystem: a modular home orchestration system that synchronizes domestic operations, energy, finance, property state, and situational awareness.

Each subdirectory is a bounded module with its own domain, state, and purpose. Keep module-local logic inside the module unless a shared contract is explicitly designed.

## Top-Level North Star

**The Ecosystem Orchestrator**

> To orchestrate a unified, event-driven Digital Twin that synchronizes domestic operations, maximizes resource efficiency (energy, finance, time), and provides total situational awareness for the home ecosystem.

## Working Rules

- Keep the event bus transient; do not use it as historical storage.
- Prefer explicit events and module contracts over cross-module imports or hidden coupling.
- Keep persistence local to the owning module unless a shared storage boundary is designed.
- Preserve alignment between root ecosystem purpose and each module’s North Star.
