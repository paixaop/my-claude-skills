# Changelog Generation

Generate user-facing release notes from completed implementation plans by cross-referencing plan features against git history.

## Input

One of:
- **Plan file path** — a completed plan (status: `implemented`) from `docs/plans/implemented/`
- **PR number** — fetches the plan from the PR body or linked issues

## Process

### 1. Locate the Plan

**From file path:** Read the plan file directly. Verify `status: implemented` in frontmatter.

**From PR number:**
```bash
# Get PR details
gh pr view <number> --json title,body,mergeCommit,baseRefName,headRefName

# Find the plan file referenced in the PR body or linked issues
# Look for docs/plans/ paths in the body
```

If the plan isn't `implemented`, warn the user: "This plan has status '<status>'. Release notes are most accurate for completed plans. Continue anyway?"

### 2. Extract Features

From the plan file, extract:
- Feature names and numbers
- Behavior descriptions (the user-facing "what it does")
- Acceptance criteria (for detail about capabilities)
- Breaking changes (look for ACs that change existing behavior, modified API contracts, changed endpoints)

### 3. Cross-Reference Git History

Find the commits associated with this plan:

```bash
# If PR number is known (from plan frontmatter)
gh pr view <pr-number> --json commits --jq '.commits[].oid'

# Or find commits by branch
# Get merge base and tip
git log --oneline <merge-base>..<merge-commit>
```

Map commits to features using the commit convention `feat(<scope>): <feature>` and `test(<scope>): e2e tests for <feature>`. Match the `<scope>` and description against plan feature names.

For each feature, collect:
- Implementation commit(s) — `feat()` commits
- Test commit(s) — `test()` commits
- Files changed count: `git diff --stat <commit>~1..<commit> | tail -1`

### 4. Generate Release Notes

Rewrite each feature's behavior description in **user-facing language**:
- Remove technical jargon where possible
- Focus on what the user can now do, not how it's implemented
- Keep each feature description to 1-3 sentences
- Group related features under a shared heading if they form a cohesive capability

### 5. Identify Breaking Changes

A change is breaking if:
- An existing API endpoint changed its contract (different request/response shape)
- An existing CLI command changed its arguments or output format
- An existing config key was renamed, removed, or changed type
- An existing behavior was intentionally altered (not bug-fixed)

Check the plan's `## Security Considerations` and feature ACs for signals. Also check git diffs for modified (not new) API contracts.

### 6. Present Draft

Show the draft release notes to the user using the [changelog-output.md](../assets/changelog-output.md) template.

Use AskQuestion: "Here are the draft release notes. Save as-is, or adjust?"
- **Save** — write to the changelog directory
- **Adjust** — user provides feedback, regenerate

### 7. Save

Write to `docs/changelog/<name>.md` where `<name>` matches the plan name (without the date prefix and `-plan` suffix).

If the user specifies a version number, use `docs/changelog/<version>.md` instead.

Create the directory if it doesn't exist: `mkdir -p docs/changelog`

## Edge Cases

- **Plan without PR:** If `pr` is blank in frontmatter, use the plan's feature descriptions only (skip git cross-reference). Note in the output: "Generated from plan only — no git history available."
- **Commits don't match features:** If a commit's scope doesn't match any feature name, list it under "Other Changes" in the technical details table.
- **Multiple plans in one PR:** If the PR references multiple plans, generate separate release notes sections for each.
