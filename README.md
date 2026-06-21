# House Project

Welcome to the Home Digital Twin workspace. This project is an orchestrated collection of lightweight, autonomous agents designed to manage and optimize a modern smart home environment.

## Modules

*   **`core`**: The Infrastructure/Kernel (Event routing & registry).
*   **_`energy`_**: The Energy Observer (Solar/Battery telemetry).
*   **`property`**: The Property Guardian (Maintenance, Sensors, IoT).
*   **`finance`**: The Finance Auditor (TillerHQ integration, expense verification).

## Architecture Principles
- **Event-Driven**: Communication occurs via transient signals on the `workgraph` bus.
- **Agent-Centric**: Each agent is self-contained and owns its own persistent state.
- **Lightweight**: Minimal overhead; no centralized heavy database.
