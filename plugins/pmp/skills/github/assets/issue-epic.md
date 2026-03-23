## Plan Reference
- **Plan:** `docs/plans/YYYY-MM-DD-<name>-plan.md`
- **Goal:** <plan goal from header>
- **Stack:** <stack from plan header>
- **Complexity:** <SIMPLE | STANDARD | COMPLEX>

## Overview
<description — from the plan's goal and security considerations>

## Feature Dependency Graph
<from the plan's Feature Dependency Graph section>
1. Feature 1: [Name] (no dependencies) → #<sub-issue>
2. Feature 2: [Name] (depends on Feature 1) → #<sub-issue>
3. Feature 3: [Name] (depends on Feature 1) → #<sub-issue>

## Sub-Issues
<!-- Native sub-issues will be linked automatically -->

## Success Criteria
<what done looks like — from plan's acceptance criteria summary>

## Definition of Done
- [ ] All sub-issues closed
- [ ] Integration tests pass
- [ ] Documentation updated
- [ ] No P0/P1 bugs outstanding

## Full Implementation Plan
<details><summary>Click to expand full plan</summary>

<!-- IMPORTANT: Inline the plan content directly here. Do NOT use $(cat ...) or
     reference local file paths — those expose the local filesystem in the
     GitHub Issue and break for other readers. Read the plan file, then paste
     its contents (everything from "## E2E Test Infrastructure" through
     "## Security Considerations") directly into this section. -->

<inline plan content here — all features, ACs, E2E tests, security considerations, test infrastructure>

</details>

## Source Context
<details><summary>Spec/Roadmap excerpt</summary>

> <relevant portion of the spec, roadmap, or brainstorm output that generated this epic — quote verbatim>

</details>

---

## What's Next

> **Next in the PMP lifecycle:** Begin implementation.
>
> | Action | Command | When to use |
> |--------|---------|-------------|
> | Start coding | `pmp:execute` | Implement the plan with code-test-fix loop |
