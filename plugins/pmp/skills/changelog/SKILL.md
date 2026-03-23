---
name: changelog
description: Use when the user says 'generate release notes', 'changelog', 'what was built', 'summarize implementation', or wants user-facing release notes from a completed plan or PR
---

# PMP: Changelog

**Announce at start:** "Using pmp:changelog to generate release notes."

Generate user-facing release notes from completed implementation plans.

**Always confirm output before saving.** Use AskQuestion. Never auto-save.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [changelog.md](references/changelog.md) and follow it completely — it contains the generation algorithm
3. Present draft release notes to the user for review
4. Save to the changelog directory

## Key References

- Generation algorithm: [changelog.md](references/changelog.md)
- Output template: [changelog-output.md](assets/changelog-output.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
