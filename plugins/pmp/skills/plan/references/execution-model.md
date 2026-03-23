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

## SSoT Compliance Rules

When the project has spec files with SSoT conventions, every plan MUST follow these rules so the executing agent maintains spec integrity:

- **Reference canonical sources** — Every feature that implements a spec-defined behavior must link to the canonical spec file (from the SSoT index). Never reference duplicates, summaries, or inline copies.
- **One concept = one file** — If a feature adds a new architectural concept, the plan must include a task to create its canonical spec file. No new concepts defined only in code comments or inline docs.
- **Update specs alongside code** — If a feature changes spec-owned behavior, the plan must include an explicit task to update the canonical spec file. Code changes without spec updates are incomplete.
- **No duplication** — Plan tasks must never instruct the agent to duplicate spec content into other files. Cross-reference with section-level links (`file.md#section`) instead.
- **Update the SSoT index** — If the plan creates or removes spec files, include a task to update `spec-index.md`.
- **Stable anchors** — If a plan task renames a heading in a spec file, it must include updating all inbound cross-reference links. Use grep to find them.
- **Section-level links** — All cross-references in plan tasks must use `file.md#section` format, not bare file links, so the executing agent links to the exact location.
- **No magic numbers** — Numeric literals in plan tasks (thresholds, limits, timeouts) must reference a named setting from the settings catalog. If the setting doesn't exist yet, include a task to add it.
