## Plan Review: [Plan Name]
<!-- Template for Plan Review output — used by references/review.md -->

### Summary
[1-2 sentence overall assessment]

### Verdict: [APPROVED | NEEDS REVISION | REJECTED]

### Issues Found

#### Critical (must fix)
- [Issue with file:line reference and specific fix]

#### Important (should fix)
- [Issue with recommendation]

#### Minor (nice to have)
- [Suggestion]

### Spec Alignment Analysis (when spec source available)
<!-- Include when the plan's Source: field points to a local spec file/directory.
     Omit entirely when spec alignment gate was skipped. -->

#### Traceability Matrix

| ID | Spec Requirement | Plan Feature(s) | Coverage | Gap |
|----|-----------------|-----------------|----------|-----|
| [REQ-1] | [requirement text] | Feature N | Covered / Partial / Missing / Contradicted | [what's missing or contradicted] |

#### Unaddressed Requirements
- [Requirement not covered by any feature — with spec file:line reference]

#### Contradictions
- [Plan statement] contradicts [spec statement] — **Recommendation:** [specific fix]

#### Ambiguities Detected
- [Spec area] is ambiguous; plan assumes [X] — **Risk:** [low/medium/high] — **Action:** [clarify with stakeholder / add assumption to plan / flag as risk]

---

### Changes by File
<!-- Groups ALL issues from every section by target file for parallel agent execution.
     Each file entry collects every finding that touches that file. -->

#### `path/to/file1.ext`

| # | Issue | Severity | Source | Action |
|---|-------|----------|--------|--------|
| 1 | [issue description] | Critical | Spec gap / Architecture / Security | [specific change needed] |
| 2 | [issue description] | Important | Spec contradiction / Completeness | [specific change needed] |

**Spec requirements:** [REQ-1, BEH-2 — list of spec items touching this file]
**Plan features:** [Feature N, Feature M]

#### `path/to/file2.ext`

| # | Issue | Severity | Source | Action |
|---|-------|----------|--------|--------|
| 1 | [issue description] | Minor | Conventions | [specific change needed] |

**Spec requirements:** [REQ-3]
**Plan features:** [Feature N]

#### Files Not in Plan but Required by Spec
- `path/to/missing.ext` — Required by [REQ-N], not listed in any feature's Affected Files

---

### Config & Migration Analysis (if applicable)
[Config changes: new/modified env vars, config keys, required vs optional]
[Schema changes: tables, columns, indexes, constraints affected]
[Migration checklist: up/down scripts, data backfill, rollback verification]

#### Config Simplification Options (if applicable)
| Current Key | Proposed Key | Reason |
|---|---|---|
| [cryptic_name] | [intuitive_name] | [why it's clearer] |
[Recommendation: accept all / accept subset / keep as-is]

#### Table Consolidation Options (if applicable)
| Option | Tables Affected | Schema After | Trade-offs |
|---|---|---|---|
| A) Keep separate | [tables] | [no change] | [pros/cons] |
| B) Merge into one | [tables → target] | [merged schema] | [pros/cons] |
| C) Hybrid | [partial merge] | [schema] | [pros/cons] |
[Recommendation with justification — user decides]

### Security Analysis (if applicable)
[STRIDE findings, attack trees, risk matrix — from security-analysis.md workflow]
[Spec change recommendations grouped by severity]

### Improved Sections
[Rewritten sections replacing weak originals]

### Missing Items
[Items the plan should include but doesn't]

### Previous Finding Resolution (re-review only)
<!-- Include this section only when re-reviewing a plan that was previously reviewed.
     Shows the status of each finding from the prior review. -->

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | [Finding description] | Critical | Resolved — [how it was addressed] |
| 2 | [Finding description] | Important | Unresolved — [still missing] |
| 3 | [Finding description] | Minor | Resolved |

---

## What's Next

> **Next in the PMP lifecycle:** Act on the review verdict.
>
> | Verdict | Action | Command | When to use |
> |---------|--------|---------|-------------|
> | **APPROVED** | Create GitHub issues | `pmp:github` | Publish plan as trackable issues before coding |
> | **APPROVED** | Start coding | `pmp:execute` | Skip issues and begin implementation directly |
> | **NEEDS REVISION** | Discuss findings | `pmp:discuss` | Walk through issues interactively to collect fixes |
> | **NEEDS REVISION** | Update plan and re-review | `pmp:plan-review` | After updating the plan, run review again |
