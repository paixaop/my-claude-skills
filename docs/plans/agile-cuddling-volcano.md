# Enhance pmp:review — Spec Alignment + File-Grouped Changes

## Context

The plan review skill currently evaluates plans for quality but never cross-references them against their source specs. This means ambiguities, missing coverage, and contradictions between spec and plan go undetected. Additionally, issues are grouped by severity but not by file, making it hard to parallelize fixes during execution.

This change adds a **Spec Alignment Gate** (reads specs, builds traceability matrix, detects gaps) and a **Changes by File** output section (groups all issues by target file for parallel agent work).

---

## Changes by File

### CREATE: `plugins/pmp/skills/review/references/spec-alignment.md`

New reference file (~120 lines) detailing the spec alignment analysis process. Modeled on the existing `security-analysis.md` pattern. Contains:

- **When to Use** — mandatory when plan's `Source:` field points to a local file/directory; skipped otherwise
- **Extract from Spec** — requirements (MUST/SHOULD/MAY), behaviors, constraints, entities
- **Extract from Plan** — per-feature behavior, affected files, ACs, API contracts
- **Traceability Matrix** — maps every spec requirement to plan feature(s), marks as Covered / Partial / Missing / Contradicted
- **Ambiguity Detection** — flag vague spec areas where the plan makes silent assumptions
- **Gold-Plating Detection** — features that don't trace back to any spec requirement
- **Affected Files Verification** — cross-check each feature's file list against what the spec implies

### MODIFY: `plugins/pmp/skills/review/references/review.md`

Current: 213 lines. After: ~260 lines.

1. **State diagram** (lines 9-44) — add `SpecAlignmentGate` between `ReadAllCode` and existing conditional gates:
   ```
   ReadAllCode --> SpecAlignmentGate : spec source available
   SpecAlignmentGate --> ConfigDataGate : plan touches config/DB
   SpecAlignmentGate --> SecurityGate : plan touches auth/endpoints
   SpecAlignmentGate --> ScoreChecklist : neither
   ```

2. **New checklist category** — insert after "Architecture & Design" (line 98), before "Security" (line 99):
   ```
   ### Spec Alignment (when spec source available)
   - [ ] Plan's `Source:` field points to a valid, readable spec file or directory
   - [ ] Every spec requirement maps to at least one plan feature
   - [ ] Every plan feature traces back to a spec requirement
   - [ ] No plan feature contradicts a spec requirement
   - [ ] Ambiguous spec areas flagged (not silently interpreted)
   - [ ] Feature "Affected Files" lists are complete per spec requirement
   - [ ] Acceptance criteria cover spec requirements (not just implementation details)
   ```

3. **New process step 2.5** — insert between "Read ALL referenced code files" and "Config & data gate":
   ```
   2.5. Spec alignment gate: If plan has a Source: field pointing to a local file/directory:
   - Read all spec files from the source
   - Build requirements-to-feature traceability matrix
   - Detect: unaddressed requirements, contradictions, silent ambiguity interpretations, incomplete Affected Files
   - Read spec-alignment.md for full process
   - If Source: missing or non-local → skip, note "Spec alignment: skipped" in output
   ```

### MODIFY: `plugins/pmp/skills/review/assets/review-output.md`

Current: 57 lines. After: ~105 lines.

1. **Add "Spec Alignment Analysis" section** — after "Issues Found", before "Config & Migration Analysis":
   - Traceability matrix table (Spec Requirement | Plan Feature(s) | Coverage | Gap)
   - Unaddressed Requirements list
   - Contradictions list
   - Ambiguities Detected list

2. **Add "Changes by File" section** — after all analysis sections, before "Improved Sections":
   - Groups ALL issues from every section by file path
   - Per file: table of issues (severity, source section, specific action needed)
   - Per file: which spec requirements and plan features touch it
   - "Files Not in Plan but Required by Spec" sub-section for missing coverage

### MODIFY: `plugins/pmp/skills/review/SKILL.md`

Current: 32 lines. After: ~38 lines.

- Add to Key References: `- Spec alignment: [spec-alignment.md](references/spec-alignment.md)`
- Update workflow step 2 description to mention spec alignment gate

### MODIFY: `plugins/pmp/.claude-plugin/plugin.json`

Version bump: `1.8.1` → `1.8.2`

---

## Execution Parallelism

| Wave | Files | Dependencies |
|------|-------|-------------|
| Wave 1 | `spec-alignment.md` (create), `review-output.md` (modify) | None — independent |
| Wave 2 | `review.md` (modify) | Depends on Wave 1 (references spec-alignment.md) |
| Wave 3 | `SKILL.md` (modify), `plugin.json` (modify) | Depends on Wave 1 |

---

## Verification

1. Read each modified file end-to-end — confirm line counts under 500
2. Verify all cross-references resolve (SKILL.md → spec-alignment.md, review.md → spec-alignment.md)
3. Dry-run: take an existing plan from `docs/plans/` that has a `Source:` field and mentally walk through the new workflow to confirm it's complete
