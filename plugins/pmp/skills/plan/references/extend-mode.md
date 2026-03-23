# Extend Mode

When a plan already exists and the user provides an updated roadmap:

## 1. Read Existing Plan

Read the plan completely.

## 2. Classify Roadmap Items

For each item in the new/updated roadmap:

- **New feature**: doesn't exist in current plan
- **Modified feature**: changes behavior of an existing feature
- **Removed feature**: no longer needed (user must confirm removal)

## 3. Apply Changes

**New features:**
- Append with next sequential feature number (implementation spec + E2E tests together)
- Add dependency links to existing features where applicable

**Modified features:**
- Update the existing feature spec in-place
- Mark with `**Updated: YYYY-MM-DD**`
- Update the E2E test cases under that feature to match the new behavior

**Removed features:**
- Mark as `**Removed: YYYY-MM-DD**` (don't delete -- audit trail)
- Mark the E2E test cases as skipped

## 4. Re-verify Coverage

Walk through every active (non-removed) feature and confirm all ACs have test cases.

## 5. Present Change Summary

Before saving, present a structured diff summary to the user for confirmation:

```
## Plan Changes: [Plan Name]

### Added
| Feature | Dependencies | ACs | E2E Tests |
|---------|-------------|-----|-----------|
| Feature N: [Name] | [deps] | [count] | [count] |

### Modified
| Feature | What Changed |
|---------|-------------|
| Feature K: [Name] | [sections changed — e.g., "AC-K.2 updated, new affected file"] |

### Removed
| Feature | Reason |
|---------|--------|
| Feature J: [Name] | [reason from user or roadmap] |

### Unchanged
- Feature 1, Feature 2, ...
```

Use AskQuestion: "Here are the planned changes. Apply them?"

## 6. Save Updated Plan

Same filename. Preserve existing frontmatter — do NOT reset `status` or timestamps from prior stages. Add a changelog section below the frontmatter/header:

```markdown
## Changelog

- **YYYY-MM-DD**: Added Feature N, Feature M. Modified Feature K. Removed Feature J.
```
