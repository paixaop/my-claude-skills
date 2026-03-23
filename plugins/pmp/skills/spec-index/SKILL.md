---
name: spec-index
description: Use when the user says 'generate spec index', 'build spec index', 'create spec index', 'update spec index', 'spec index', 'ssot index', or has spec files that need an ownership registry so agents know which file owns which concept
---

# PMP: Spec Index Generator

**Announce at start:** "Using pmp:spec-index to generate the spec ownership registry."

Scans a spec directory, reads every file's SSoT header and summary block, and produces a machine-readable index mapping each file to the concepts it owns. Other PMP sub-commands read this index to locate canonical sources without scanning all spec files.

**Always confirm before writing.** Use AskQuestion. Never auto-write.

## When NOT to Use

- Specs don't follow SSoT conventions yet → use pmp:arc42 first to restructure
- Want to review spec quality → use pmp:spec-review instead
- Want to reorganize specs → use pmp:arc42 instead

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [spec-index-generator.md](references/spec-index-generator.md) and follow it completely
3. Present the generated index to the user for review
4. Save to the spec directory root as `spec-index.md`

## Key References

- Generator algorithm: [spec-index-generator.md](references/spec-index-generator.md)
- SSoT formatting rules: [single-source-of-truth.md](../pmp/references/single-source-of-truth.md)

## Shared Resources

- **BACKGROUND:** [overview.md](../pmp/references/overview.md) for lifecycle context
