# Decompose Plan into Phases

Break an existing implementation plan into dependency-ordered phases. This is the standalone version of the auto-phasing step in plan generation (see [generate-plans.md](../../plan/references/generate-plans.md) step 4b).

## Input

A plan file path. The plan must have a `## Feature Dependency Graph` section.

## Process

### 1. Read the Plan

Read the plan completely. Extract:
- All features with their names and numbers
- The dependency graph (which features depend on which)
- Feature count — if fewer than 5 features, ask the user: "This plan has N features. Phasing is most useful for 5+. Continue anyway?"

### 2. Build Dependency Layers

Group features into layers using topological ordering:

- **Layer 0:** Features with no dependencies
- **Layer 1:** Features depending only on Layer 0 features
- **Layer N:** Features depending only on features in layers 0 through N-1

### 3. Apply Cohesion Grouping

Within each layer, check for cohesion opportunities:
- Features touching the same files or domain area → keep together
- If a layer has more than 5 features → split into sub-phases by cohesion
- Adjacent layers with only 1-2 features each → consider merging if dependencies allow

### 4. Name Phases

Give each phase a descriptive name reflecting its purpose:
- "Foundation" — base infrastructure, models, core setup
- "Core Logic" — primary business logic and API endpoints
- "User-Facing" — UI components, user flows
- "Polish" — error handling improvements, edge cases, optimization

Use domain-specific names when they're more descriptive.

### 5. Define Entry/Exit Criteria

For each phase:
- **Entry:** The prior phase's exit criteria are met (Phase 1 entry: branch created, E2E infrastructure ready)
- **Exit:** All features in this phase have passing E2E tests + CI green

### 6. Present to User

Show the proposed phasing:

```
## Proposed Phases

### Phase 1: [Name] (N features)
Features: Feature 1, Feature 2, Feature 3
Entry: Branch created, E2E infrastructure ready
Exit: All Phase 1 E2E tests pass, CI green

### Phase 2: [Name] (N features)
Features: Feature 4, Feature 5
Depends on: Phase 1
Entry: Phase 1 exit criteria met
Exit: All Phase 2 E2E tests pass, CI green
```

Use AskQuestion: "Here's the proposed phasing. Adjust or apply?"
- **Apply** — insert into the plan
- **Adjust** — user suggests changes, re-present

### 7. Insert into Plan

Insert the `## Phases` section between `## Feature Dependency Graph` and the first feature section. Use the template from [plan.md](../../plan/assets/plan.md).

## Phase Count Guidelines

| Features | Typical Phases |
|----------|---------------|
| 5-7      | 2-3           |
| 8-12     | 3-4           |
| 12+      | 4+            |

Minimize phase count. Don't create phases for the sake of it — each phase should represent a meaningful milestone where the system is in a testable, stable state.

## Rules

- Features within a phase MUST have no cross-dependencies (can be parallelized)
- Features in Phase N+1 MUST depend on at least one feature in Phase N (or earlier)
- Phase boundaries align with execution batch boundaries (see [execute-loop.md](../../execute/references/execute-loop.md))
- Never split tightly coupled features across phases — if Feature A and Feature B always need to be tested together, they belong in the same phase
