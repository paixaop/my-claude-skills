# Plan: Add "Unspecified References" Criterion to Spec-Review Suite

## Context

The spec-review suite's implementability assessment (11 criteria) misses a critical class of gaps: items **mentioned in prose but never formally defined**. For example, a spec says "configure the timeout" but never specifies the config key name, type, default, or valid range. These "phantom specifications" are distinct from ambiguities (Criterion 10) — the spec is explicit that something *exists* but never provides the concrete definition needed to implement it. A recent review demonstrated this gap by finding many unspecified items the suite failed to catch.

## Changes

### 1. Add Criterion 12 to spec-implementability reference

**File:** `plugins/pmp/skills/spec-implementability/references/spec-implementability.md` (190 → ~218 lines)

Insert after Criterion 11 (line 183), before `## Output` (line 186):

**Criterion 12: Unspecified References** — scan the entire spec for items mentioned in prose but never formally defined. Categories to scan:

- **Config settings / env vars** — no name, type, default, valid range
- **Metrics / telemetry** — no metric name, dimensions, units, thresholds
- **Error codes / status codes** — no error catalog or code-to-message mapping
- **Roles / permissions** — no role enumeration or permission matrix
- **Events / signals / hooks** — no event name, schema, or delivery guarantees
- **Thresholds / limits / quotas** — no concrete values or enforcement behavior
- **Feature flags** — no flag names, default states, or management system
- **CLI commands / flags** — no command syntax or usage
- **Status / state enumerations** — no enumeration of valid states or transitions

For each found: quote the prose, classify by category, assess severity (BLOCKER/HIGH/MEDIUM/LOW), list missing attributes.

Also update cross-reference bullets (line 27-29) to add:
`- **spec-architecture + spec-operations findings** → inform criterion 12`

### 2. Add Criterion 12 output to standalone template

**File:** `plugins/pmp/skills/spec-implementability/assets/spec-implementability-output.md` (195 → ~209 lines)

Insert after Criterion 11 table (line 150), before `## Gap List` (line 153):

```
### 12. Unspecified References

[detailed reasoning — list each phantom reference found with spec quote]

| ID | Category | Prose Mention | Missing Definition | Location | Severity |
|----|----------|---------------|--------------------|----------|----------|
| UR-NNN | config / metric / error code / role / event / threshold / flag / CLI / enum | [quote] | [missing attributes] | [file/section] | BLOCKER / HIGH / MEDIUM / LOW |
```

### 3. Condense consolidated template to make room

**File:** `plugins/pmp/skills/spec-review/assets/spec-review-output.md` (491 lines, limit 500)

Remove ~8 redundant lines to create room:
- Remove blank lines between `---` separators and `####` headings in the implementability section (lines ~340, ~348, ~356, ~364 area)
- Remove redundant example rows from criteria 5-9 tables where the pattern is already established by criteria 1-4

### 4. Add Criterion 12 output to consolidated template

**File:** `plugins/pmp/skills/spec-review/assets/spec-review-output.md`

Insert after Criterion 11 block (after line ~431), before Gap List section. Same table format as step 2.

Target: ≤ 495 lines after condensation + addition.

### 5. Update "11-criteria" → "12-criteria" references

6 occurrences across 5 files (simple string replacement):

| File | Line |
|------|------|
| `plugins/pmp/skills/spec-implementability/SKILL.md` | 3 (description) |
| `plugins/pmp/skills/spec-implementability/SKILL.md` | 8 (body) |
| `plugins/pmp/skills/pmp/SKILL.md` | 29 |
| `plugins/pmp/skills/pmp/references/overview.md` | 380 |
| `plugins/pmp/skills/spec-review/SKILL.md` | 21 |
| `plugins/pmp/skills/spec-review/references/spec-review.md` | 104 |

### 6. Bump plugin version

**File:** `plugins/pmp/.claude-plugin/plugin.json`

`1.8.3` → `1.8.4`

## Key Design Decisions

- **Why Criterion 12 (not renumbering)?** Avoids touching Criterion 11 content. Only the count string "11-criteria" → "12-criteria" changes.
- **Why in spec-implementability?** It's the completeness gatekeeper with the "ruthless production gatekeeper" mindset. Phantom references are a completeness problem.
- **Distinction from Criterion 10:** Criterion 10 asks "is this *clear*?" (ambiguities, contradictions). Criterion 12 asks "is this *defined*?" (mentioned but unspecified).
- **Overlap with other sub-commands is intentional:** spec-operations may catch missing metrics, spec-architecture may catch config issues. Criterion 12 provides a *systematic* scan rather than relying on incidental discovery. The orchestrator deduplicates.

## Verification

1. Count lines in all modified files — must be ≤ 500
2. Grep for "11-criteria" — should return 0 results after changes
3. Run `/pmp:spec-review` on a test spec corpus and verify Criterion 12 appears in the output with phantom references identified
