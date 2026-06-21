# Core Development Standards

## Responsibility
Maintain the transient, high-speed routing of signals without introducing stateful bloat or indexing heavy payloads.

## Guidelines
1.  **Transient Only**: Do not attempt to persist event history in this module.
2.  **Schema Integrity**: Ensure events published by agents follow a consistent, lightweight structure.
3.  **Registry Management**: Handle agent registration/deregistration with minimal latency.
