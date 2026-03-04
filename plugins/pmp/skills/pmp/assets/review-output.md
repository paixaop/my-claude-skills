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
