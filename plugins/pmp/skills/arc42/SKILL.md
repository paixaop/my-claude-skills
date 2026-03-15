---
name: arc42
description: "Reorganize spec and architecture files into arc42 standard structure — consolidates scattered content into 12 arc42 section directories with one file per concern. Use when the user says 'organize specs', 'arc42', 'consolidate docs', 'clean up specs', 'reorganize architecture', 'merge spec files', 'deduplicate specs', or has spec files that grew disorganized over multiple revisions and reviews. Reads all files, classifies content into arc42 sections, identifies duplicates/overlaps/contradictions, proposes a new structure, and executes the reorganization after user approval. Standalone workflow — does not feed into plan generation or execution."
---

# PMP: Arc42

Reorganize spec/architecture files into the arc42 standard structure. Reads the corpus, classifies content into arc42's 12 sections, identifies duplicates and contradictions, proposes a new directory structure, and executes the reorganization after user approval.

Standalone workflow — does not feed into plan generation or execution.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) ONLY for parallel file reading when the corpus is large. All analysis runs in the main controller context — do not spawn agents for analysis phases. Track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md). Check `docs/.cache/arc42/manifest.json`. Load cached summaries for unchanged files, only read changed/new files in full. This is mandatory, not optional.
3. Read [arc42.md](references/arc42.md) and follow it completely — it contains the full reorganization algorithm
4. Present reorganization proposal to user for approval/adjustment
5. Execute approved reorganization
6. Verify no content was lost

## Key References

- Arc42 section guidance and tips: [arc42-guide.md](references/arc42-guide.md)
- Reorganization report template: [reorganization-report.md](assets/reorganization-report.md)

## Shared Resources

- Configuration & constants: [config.md](../pmp/config.md)
- Analysis cache: [analysis-cache.md](../pmp/references/analysis-cache.md)
- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
