# Discuss Skill: Proper Plan Naming + All Findings in Plan

## Context

When running spec-review analysis multiple times, acknowledged and non-fix findings keep resurfacing because:
1. The discuss skill only puts "Fix" findings as Features and "Deferred" in the plan — Acknowledged/Skipped findings are recorded nowhere persistent
2. The spec-review skill re-analyzes specs from scratch without checking if findings were previously acknowledged
3. Plan files sometimes get generic Claude Code names (like `quirky-strolling-harbor.md`) instead of the prescribed `YYYY-MM-DD-<name>-findings-plan.md`

## Approach

Embed GitHub-flavored blockquote alerts into spec files for Acknowledged/Skipped findings during discuss Step 4, using three alert types based on severity and decision:
- `> [!WARNING]` for **Acknowledged** findings
- `> [!NOTE]` for **Skipped** findings
- `> [!CAUTION]` for **Critical severity Acknowledged** findings

Spec-review then detects these alerts and provides context instead of blindly re-raising.

## Changes

### 1. `discuss/assets/discuss-plan.md` — Add audit trail table

Add `## Acknowledged & Skipped` section after `## Deferred`:

```markdown
## Acknowledged & Skipped

Findings resolved during discussion without spec changes. Annotations added to spec files.

| Finding | Severity | Decision | Reason | Annotated In |
|---------|----------|----------|--------|--------------|
| [title] | [severity] | Acknowledged / Skipped | [reason] | [spec file path] |
```

### 2. `discuss/references/discuss.md` — Rewrite Step 4

**Replace Step 4 (lines 189-204) with three sub-steps:**

**4a. Annotate Spec Files** (for Acknowledged and Skipped findings):
- For each finding, locate the relevant spec file and section
- Insert a GitHub blockquote alert **immediately after** the relevant section:

  For **Critical Acknowledged**:
  ```markdown
  > [!CAUTION]
  > **Reviewed 2026-03-15:** [finding title] — **Acknowledged:** [reason]
  ```

  For **non-Critical Acknowledged**:
  ```markdown
  > [!WARNING]
  > **Reviewed 2026-03-15:** [finding title] — **Acknowledged:** [reason]
  ```

  For **Skipped**:
  ```markdown
  > [!NOTE]
  > **Reviewed 2026-03-15:** [finding title] — **Skipped:** [reason or "No reason given"]
  ```

- If finding refers to missing content, place at top of most relevant spec file after title

**4b. Generate Plan File** (if any findings marked Fix):
- Use Write tool with explicit path: `docs/plans/YYYY-MM-DD-<review-name>-findings-plan.md`
- Fix findings become Features (spec changes)
- Deferred in `## Deferred` section
- Acknowledged/Skipped in `## Acknowledged & Skipped` section (audit trail)
- Add CRITICAL instruction: DO NOT use Claude Code auto-generated names, DO NOT use plan mode — use Write tool directly

**4c. No-Fix Path** (all findings Acknowledged/Skipped/Deferred):
- Still annotate spec files per 4a
- Skip plan file generation
- Report: "All findings resolved without spec changes. Annotations added to spec files."

**Also update:**
- State diagram (lines 36-38): `Summary --> AnnotateAndPlan` instead of `Summary --> GeneratePlan`
- Trigger condition: change from "if findings marked Fix" to "if any findings were discussed"
- Constraints: add "DO NOT use Claude Code plan mode or auto-generated filenames"

### 3. `spec-review/references/spec-review.md` — Detect prior annotations

**Add new step in Phase 0 Discovery (between steps 2 and 3, shifting existing 3-4):**

```markdown
3. **Collect prior review annotations**
   - Scan spec files for `> [!WARNING]`, `> [!NOTE]`, and `> [!CAUTION]` blockquotes containing "**Reviewed"
   - Parse into: { date, finding_title, decision, reason, file }
   - Build prior decisions registry
   - Report: `Found N prior review annotations across M files`
```

**Add subsection "Using Prior Review Annotations":**
- During analysis phases, check findings against prior annotations
- `[!CAUTION]` or `[!WARNING]` (Acknowledged) match: include with `[Previously acknowledged YYYY-MM-DD: "reason"]` prefix, reduce severity one level (Critical stays Critical)
- `[!NOTE]` (Skipped) match: include normally with `[Previously skipped YYYY-MM-DD]` note
- No match: report normally

### 4. `execute/references/execute-loop.md` — Minor doc update

In Completion step 9 (line 592), after the Acknowledged bullet, add note:
```
Note: spec files were annotated with GitHub blockquote alerts (WARNING/NOTE/CAUTION) during discussion
```

## Files to Modify

| File | Change Type |
|------|------------|
| [discuss-plan.md](../../plugins/pmp/skills/discuss/assets/discuss-plan.md) | Add Acknowledged & Skipped table |
| [discuss.md](../../plugins/pmp/skills/discuss/references/discuss.md) | Rewrite Step 4, update state diagram, update constraints |
| [spec-review.md](../../plugins/pmp/skills/spec-review/references/spec-review.md) | Add annotation detection in Phase 0, add annotation matching rules |
| [execute-loop.md](../../plugins/pmp/skills/execute/references/execute-loop.md) | Minor documentation note |

## Verification

1. Run `/pmp:spec-review` on specs with known findings
2. Run `/pmp:discuss` on the review — acknowledge some, fix some, skip some
3. Verify plan file has proper name (`YYYY-MM-DD-*-findings-plan.md`)
4. Verify spec files contain GitHub alert annotations (`[!WARNING]`/`[!NOTE]`/`[!CAUTION]`) for acknowledged/skipped findings
5. Verify plan includes all findings (Features for fix, tables for acknowledged/skipped/deferred)
6. Run `/pmp:spec-review` again on same specs
7. Verify previously acknowledged findings appear with `[Previously acknowledged]` context and reduced severity
