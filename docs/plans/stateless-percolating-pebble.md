---
status: implemented
completed_at: "2026-03-15T00:00:00Z"
created_at: "2026-03-15T00:00:00Z"
---

# Plan: Arc42 file splitting + Execute plan-to-PR link

## Context

Two independent improvements to PMP skills:

1. **Arc42 large file splitting** — The arc42 skill currently creates one file per concern with no size limit. Large concerns (e.g., a 1200-line authentication spec) produce unwieldy files. Files exceeding 500 lines should be split into multiple files by sub-heading.

2. **Execute plan archival in PR** — After creating a PR, the execute loop should commit the archived plan file to the branch, push it, and update the PR description with a link to the plan. This creates a permanent reference from PR → plan.

---

## Change 1: Arc42 file splitting (>600 lines)

### Files to modify

- [arc42.md](../../plugins/pmp/skills/arc42/references/arc42.md) — main algorithm
- [reorganization-report.md](../../plugins/pmp/skills/arc42/assets/reorganization-report.md) — proposal template

### Edits to `arc42.md`

**1a. Add principle (after line 17)**

Add new principle 6 after "Simplify structure":
```
6. **File size limit** — no concern file should exceed 500 lines; split large concerns into multiple files by sub-heading
```
Renumber existing principle 6 (Audit trail) → 7.

**1b. Add `Est. Lines` to section assignment table (Phase 2a, ~line 130)**

Add column to the table:
```
| Source File | Source Section | Arc42 Section | Concern | Word Count | Est. Lines |
```

Add after "Report the assignment table to the user.":
```
Estimate lines per concern: `Est. Lines ≈ Word Count / 5` (accounts for headings, lists, code blocks, and whitespace).
```

**1c. Add "File Splitting Rules" sub-section (Phase 2d, after line 189)**

Insert after the existing Rules block, before Phase 2e:

```markdown
#### File Splitting Rules

When a concern's estimated line count exceeds **500 lines**, split it into multiple files within the same section directory:

1. **Identify split boundaries** — use the concern's top-level sub-headings (`##` within the concern) as split points. Each sub-heading group becomes its own file.

2. **Naming convention** — flat files with pattern `<concern>-<sub-topic>.md`:
   ```
   05-building-block-view/
   ├── README.md
   ├── authentication-overview.md        ← split from authentication
   ├── authentication-flows.md           ← split
   ├── authentication-token-management.md ← split
   ├── data-model.md                     ← not split (under 600 lines)
   └── api-gateway.md
   ```

3. **Overview file** — the first split file (`<concern>-overview.md`) starts with a navigation index:
   ```markdown
   ## Authentication

   > This concern is split across multiple files:
   > - [Overview](authentication-overview.md) (this file)
   > - [Authentication Flows](authentication-flows.md)
   > - [Token Management](authentication-token-management.md)
   ```

4. **Section README grouping** — list split files grouped under their parent concern:
   ```markdown
   ## Contents
   - **Authentication**
     - [Overview](authentication-overview.md)
     - [Flows](authentication-flows.md)
     - [Token Management](authentication-token-management.md)
   - [Data Model](data-model.md)
   ```

5. **Minimum split size** — don't create split files under 50 lines. Merge small sub-heading sections with adjacent ones.

6. **Fallback** — if the concern has no `##` sub-headings, split at ~400-line paragraph boundaries: `<concern>-part-1.md`, `<concern>-part-2.md`.
```

**1d. Add cross-reference rows (after line 210)**

Add to the cross-reference format table:
```
| Split file | Other split file in **same** concern | `[Sub-Topic](concern-sub-topic.md)` |
| Split file | Overview file of same concern | `[← Concern Overview](concern-overview.md)` |
```

**1e. Update Phase 4b (after line 299)**

Add step 7 after "Update internal cross-references":
```markdown
7. **Handle split files**: When writing a concern marked for splitting:
   - Create each split file with standard header, provenance, content, annotations, and cross-references
   - Add the concern navigation index to the overview file
   - Each split file's provenance lists only the source sections it contains
```

**1f. Add metric to Phase 5 summary report (line ~365)**

Add row to the verification summary table:
```
| Concerns split (>500 lines) | S |
```

### Edits to `reorganization-report.md`

**Update the Section Assignment Table** (~line 21) — add `Est. Lines` column:
```
| Source File | Source Section | Arc42 Section | Concern | Word Count | Est. Lines |
```

**Update the Proposed Structure tree** (~line 29) — show split example:
```
<target-dir>/
├── README.md
├── NN-section-name/
│   ├── README.md
│   ├── concern-a.md
│   ├── concern-b-overview.md        ← split (est. 800 lines)
│   ├── concern-b-sub-topic.md       ← split
│   └── concern-c.md
└── ...
```

**Update the structure summary table** (~line 39) — add `Split?` column:
```
| # | Section Directory | Concern Files | Source Files | Estimated Words | Split? |
```

---

## Change 2: Execute — commit archived plan + update PR

### File to modify

- [execute-loop.md](../../plugins/pmp/skills/execute/references/execute-loop.md) — Completion section

### Edits

**2a. Update Completion state diagram** (lines 99-113)

Add new states after `ArchivePlan`:
```
ArchivePlan --> CommitPlanArchive
CommitPlanArchive --> UpdatePRWithPlan : PR was created
CommitPlanArchive --> UpdateReview : no PR
UpdatePRWithPlan --> UpdateReview : if source_review exists
UpdatePRWithPlan --> OfferRelease : no source_review
```

**2b. Add new steps 9-10 to Completion** (after current step 8, ~line 588)

Insert after "Archive the plan" (step 8), before "Update source review file":

```markdown
9. **Commit and push plan archive:**
   ```bash
   git add docs/plans/ docs/plans/implemented/
   git commit -m "chore(<scope>): archive plan <plan-name>"
   git push
   ```

10. **Update PR with plan link (if PR was created):**
    Append a "Plan" section to the PR body linking to the archived plan file:
    ```bash
    gh pr edit <pr-number> --body "$(gh pr view <pr-number> --json body -q .body)

    ## Plan

    [<plan-name>](docs/plans/implemented/<plan-file>.md)"
    ```
    Skip this step if the user declined PR creation.
```

Renumber subsequent steps: current step 9 → 11, step 10 → 12, step 11 → 13.

---

## Verification

- Read the modified `arc42.md` and confirm:
  - Principle 6 mentions 500-line limit
  - Phase 2a table has `Est. Lines` column
  - Phase 2d has "File Splitting Rules" sub-section with 6 rules
  - Cross-reference table has 2 new rows for split files
  - Phase 4b has step 7 for split file handling
  - Phase 5 summary has split metric row
- Read the modified `reorganization-report.md` and confirm:
  - Section assignment table has `Est. Lines` column
  - Structure tree shows split file example
  - Structure summary table has `Split?` column
- Read the modified `execute-loop.md` and confirm:
  - Completion diagram has `CommitPlanArchive` and `UpdatePRWithPlan` states
  - Steps 9-10 exist (commit+push archive, update PR body)
  - Steps are numbered correctly (13 total)
  - The `gh pr edit` command appends to existing body, doesn't overwrite
