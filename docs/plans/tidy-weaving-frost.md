# PMP v1.8.0 — Depth Enhancements: Decomposition, Comparison, Re-review, Changelog

## Context

PMP v1.7.0 has a complete planning-to-execution lifecycle, but lacks depth in several areas: large plans aren't broken into phases, approach comparisons during brainstorm are conversational rather than structured, re-reviewing updated plans starts from scratch, and there's no way to generate user-facing release notes from completed plans. These four enhancements address those gaps.

---

## Feature 1: Plan Decomposition (enhance `/pmp:plan` + new `/pmp:decompose`)

### Auto-phasing in `/pmp:plan`

After generating features and the dependency graph, if the plan has **5+ features**, auto-group them into phases:

- Features within a phase have no cross-dependencies (can parallelize)
- Features in Phase N+1 depend on at least one feature in Phase N
- Each phase gets entry/exit criteria derived from its features' ACs
- Phase count is minimized

New `## Phases` section in plan template, between `## Feature Dependency Graph` and the first feature:

```markdown
## Phases

### Phase 1: [Name]
**Features:** Feature 1, Feature 2, Feature 3
**Entry:** Branch created, E2E infrastructure ready
**Exit:** All Phase 1 E2E tests pass, CI green

### Phase 2: [Name]
**Features:** Feature 4, Feature 5
**Depends on:** Phase 1
**Entry:** Phase 1 exit criteria met
**Exit:** All Phase 2 E2E tests pass, CI green
```

Execute loop uses phase boundaries as batch boundaries instead of arbitrary 3-feature cutoff when phases exist.

### Standalone `/pmp:decompose`

New sub-skill. Takes an existing plan file path, reads dependency graph and features, proposes phase groupings, asks user to confirm/adjust, inserts `## Phases` section.

**Files to modify:**
- `plugins/pmp/skills/plan/assets/plan.md` — add `## Phases` section template
- `plugins/pmp/skills/plan/references/generate-plans.md` — add auto-phasing step after dependency graph generation
- `plugins/pmp/skills/execute/references/execute-loop.md` — use phase boundaries as batch boundaries when present

**Files to create:**
- `plugins/pmp/skills/decompose/SKILL.md` — sub-skill entry point
- `plugins/pmp/skills/decompose/references/decompose.md` — phasing algorithm

---

## Feature 2: Plan Comparison (enhance `/pmp:brainstorm`)

Add structured comparison matrix when proposing approaches. Dynamic dimensions (4-6 most relevant):

```markdown
### Approach Comparison

| Dimension        | A: [Name]  | B: [Name]  | C: [Name]  |
|------------------|------------|------------|------------|
| Complexity       | Low        | High       | Medium     |
| Time to deliver  | Fast       | Slow       | Medium     |
| ...              | ...        | ...        | ...        |
| **Recommendation** | —        | —          | **Yes**    |

**Why [recommended]:** [1-2 sentences]
```

Available dimensions (pick 4-6 per comparison): Complexity, Time to deliver, Scalability, Maintainability, Risk, Security surface, Performance, Testability, Migration effort, Team familiarity.

Comparison table is included in the design doc output and carried forward to plan generation.

**Files to modify:**
- `plugins/pmp/skills/brainstorm/references/brainstorm.md` — add comparison format in "Proposing Approaches" section
- `plugins/pmp/skills/brainstorm/assets/design-doc.md` — add comparison table section

---

## Feature 3: Review Re-run (enhance `/pmp:review`)

When `reviewed_at` is already set in plan frontmatter, enter re-review mode:

1. **Load previous review** from `docs/reviews/` matching plan name
2. **Diff the plan** via git against the commit at `reviewed_at`
3. **Classify changes:** new features (full review), modified features (targeted), unchanged (skip)
4. **Check finding resolution:** was each previous finding's flagged section changed?
5. **Security gate always re-runs fully** (never carried forward)
6. **Produce delta review** with a `## Previous Finding Resolution` table:

```markdown
## Previous Finding Resolution

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | Missing input validation | Critical | Resolved |
| 2 | No rollback plan | Important | Unresolved |
```

Review files use consistent naming: `YYYY-MM-DD-<plan-name>-review.md`

**Files to modify:**
- `plugins/pmp/skills/review/references/review.md` — add re-review detection at top of process, delta review logic
- `plugins/pmp/skills/review/assets/review-output.md` — add "Previous Finding Resolution" section template

---

## Feature 4: Release Notes / Changelog (new `/pmp:changelog`)

New sub-skill. Input: completed plan file (status: `implemented`) or PR number.

### Workflow
1. Read plan — extract features with descriptions and ACs
2. Cross-reference git — commits between branch point and merge via `feat()/test()` conventions
3. Map features to commits using scope matching
4. Generate user-facing release notes grouped by feature
5. Flag breaking changes explicitly
6. Write to `docs/changelog/<version>.md` (configurable)

### Output template
```markdown
# Release Notes — [Feature Set Name]

**Date:** YYYY-MM-DD
**PR:** #N

## What's New

### [Feature Name]
[User-facing description rewritten from plan]

## Breaking Changes
- [If any]

## Technical Details
| Feature | Commits | Files Changed |
|---------|---------|---------------|
| [Name]  | abc1234 | 8             |
```

### Execute integration
At end of Completion phase, offer: "Want me to generate release notes?" If yes, run inline.

**Files to create:**
- `plugins/pmp/skills/changelog/SKILL.md` — sub-skill entry point
- `plugins/pmp/skills/changelog/references/changelog.md` — generation algorithm
- `plugins/pmp/skills/changelog/assets/changelog-output.md` — output template

**Files to modify:**
- `plugins/pmp/skills/execute/references/execute-loop.md` — add changelog offer in Completion
- `plugins/pmp/skills/pmp/SKILL.md` — add routing for `/pmp:changelog` and `/pmp:decompose`
- `plugins/pmp/skills/pmp/config.md` — add changelog/decompose file paths, review naming convention
- `.claude-plugin/plugin.json` — register new sub-skills, bump version to 1.8.0
- `plugins/pmp/skills/pmp/references/overview.md` — update lifecycle diagram and changelog

---

## Implementation Order

1. **Plan Comparison** (smallest change — 2 files modified)
2. **Review Re-run** (2 files modified)
3. **Plan Decomposition** (3 files modified + 2 new files)
4. **Release Notes / Changelog** (4 files modified + 3 new files)

Each feature is independent and can be implemented, tested, and committed separately.

---

## Verification

After implementation:
1. **Comparison:** Run `/pmp:brainstorm` on a test idea — verify comparison table appears and flows into design doc
2. **Re-review:** Create a plan, review it, modify the plan, review again — verify delta review with finding resolution table
3. **Decomposition:** Create a plan with 6+ features — verify auto-phasing triggers; run `/pmp:decompose` on an existing plan — verify phases inserted
4. **Changelog:** Run `/pmp:changelog` on a completed plan in `docs/plans/implemented/` — verify release notes generated with feature-to-commit mapping
5. **Plugin registration:** Verify `/pmp:changelog` and `/pmp:decompose` are routable via the PMP router
