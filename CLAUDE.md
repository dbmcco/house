# House Development Standards

## Project Context
This workspace is part of the larger `experiments` ecosystem. All development should follow the overarching principles of autonomy, lightweight implementation, and agent-centricity.

## Workflow
1.  **Design First**: Always use the `brainstorming` skill before implementing new features or modules.
2.  **Agent Isolation**: When working in a sub-module (e.g., `energy`), ensure changes do not leak into other modules' state or logic.
3.  **Event Integrity**: Respect the transient nature of the eventbus. Never use it for historical data storage.

## Tech Stack
- **Runtime**: Node.js / Python / Shell (as needed for specific agents).
- **Communication**: `workgraph` Eventbus (Transient).
- **Persistence**: Localized SQLite or JSON/CSV within each agent's directory.
- **Financial Integration**: TillerHQ / Founder Finance.
