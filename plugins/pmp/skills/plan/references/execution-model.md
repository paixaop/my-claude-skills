# Orchestrator Agent Execution Model

Plans are structured for a three-tier execution model:

```
Controller (session, max 3 features per batch)
  └── Feature Orchestrator Agent (one per feature)
      ├── Dispatches single-file task agents (parallel where dependencies allow)
      ├── Waits for dependency resolution, dispatches next wave
      ├── After all impl tasks: runs spec review inline
      ├── Dispatches test task agents (parallel where independent)
      ├── Verifies Red-Green-Refactor: each test MUST fail before passing
      ├── After all tests: runs code quality review inline
      └── Reports structured summary to controller
```

The orchestrator does NOT implement code — it only dispatches, reviews, and coordinates. Implementation agents are fresh subagents per task.

Features with no mutual dependencies (per feature dependency matrix) execute as parallel orchestrators.
