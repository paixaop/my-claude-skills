# Analysis State Cache for PMP Skills

## Context

When the spec-review skill (or any PMP skill that reads spec files) runs, it reads **all specification files completely upfront** in Phase 0 (Discovery) before beginning analysis. For large architectures with many spec files, this fills the LLM context window quickly and must be repeated every run — even if no files have changed.

**Goal:** Design a per-file digest cache that stores structured summaries of spec files, detects file changes via content hashing, and lets skills load only summaries (not full files) on subsequent runs when specs haven't changed.

## Design: Per-File Digest Cache

### Cache Structure

```
docs/.cache/<skill-name>/
├── manifest.json           # File → hash + summary mapping
└── summaries/
    ├── <sanitized-filename-1>.md   # Structured summary
    ├── <sanitized-filename-2>.md   # Structured summary
    └── ...
```

**manifest.json schema:**
```json
{
  "version": 1,
  "skill": "spec-review",
  "source_dir": "specs/",
  "created_at": "2026-03-14T10:00:00Z",
  "updated_at": "2026-03-14T12:00:00Z",
  "files": {
    "specs/architecture.md": {
      "sha256": "abc123...",
      "summary_file": "summaries/specs--architecture.md",
      "extracted_at": "2026-03-14T10:00:00Z",
      "word_count": 3200
    }
  }
}
```

**Sanitized filename:** Replace `/` with `--`, drop file extension, append `.md`. Example: `specs/auth/oauth.md` → `specs--auth--oauth.md`.

### Summary File Format

Each summary file is a structured Markdown extraction — not a free-form summary. The structure varies by skill:

**For spec-review:**
```markdown
---
source: specs/architecture.md
sha256: abc123...
extracted_at: 2026-03-14T10:00:00Z
---

## Components
- [component list with classification: data plane, control plane, etc.]

## Key Decisions
- [architectural decisions, trade-offs, constraints]

## External Dependencies
- [APIs, services, databases referenced]

## State & Data Flow
- [state management, data flow patterns]

## Cross-References
- [references to other spec files]

## Security Boundaries
- [trust boundaries, auth mechanisms]

## Terms & Identifiers
- [key terms, field names, identifiers defined in this file]
```

**For other skills (plan, review):** The summary structure should be defined per-skill based on what that skill needs from its input files. The cache mechanism is generic; the extraction template is skill-specific.

### Cache Lifecycle

#### First Run (Cold Cache)
1. Skill checks for `docs/.cache/<skill>/manifest.json`
2. Not found → full read of all spec files (existing behavior)
3. After reading each file, produce a structured summary and write to `docs/.cache/<skill>/summaries/`
4. Write `manifest.json` with all file hashes
5. Proceed with analysis using the full context (first run is unchanged)

#### Subsequent Run (Warm Cache)
1. Read `manifest.json`
2. List current spec files in source directory
3. For each file, compute SHA256 and compare to manifest:
   - **Unchanged:** Skip reading the source file
   - **Changed:** Read the source file, regenerate summary, update manifest
   - **New file:** Read, generate summary, add to manifest
   - **Deleted file:** Remove from manifest and delete summary
4. Load **only the summaries** into context (not full files)
5. If any files changed, note which ones so the analysis can focus there
6. Proceed with analysis

#### Cache Invalidation
- **Automatic:** File content hash mismatch triggers re-read of that file only
- **Manual:** User deletes `docs/.cache/<skill>/` directory to force full refresh
- **No time-based expiry** — content hashing is sufficient

### Integration Points

#### config.md Addition
Add to the File Paths table:
```
| Cache directory | `docs/.cache/` | All skills (opt-in) |
```

#### .gitignore
Add `docs/.cache/` to `.gitignore` — cached summaries are derived data, not source.

#### Spec-Review Phase 0 Modification
The Discovery phase gains a **cache check** step before "Read all documents":

```
Phase 0: Discovery
1. Map the specification corpus (unchanged)
2. Classify documents (unchanged)
3. **Check cache**
   a. If manifest exists and all hashes match → load summaries, skip full read
   b. If manifest exists but some hashes differ → read only changed files, update cache
   c. If no manifest → full read (existing behavior), then build cache
4. Read remaining documents (only uncached/changed files)
```

#### Other Skills Opt-In
Any skill that reads input files upfront can adopt this by:
1. Defining a summary extraction template (what fields to extract per file)
2. Adding the cache check step to their discovery/input phase
3. Using the same `docs/.cache/<skill>/` directory structure

### Critical Files to Modify

| File | Change |
|------|--------|
| [config.md](plugins/pmp/skills/pmp/config.md) | Add cache directory path constant |
| [spec-review.md](plugins/pmp/skills/spec-review/references/spec-review.md) | Add cache check to Phase 0 |
| `.gitignore` | Add `docs/.cache/` |
| New: `plugins/pmp/skills/pmp/references/analysis-cache.md` | Cache mechanism reference doc (reusable across skills) |

### Verification

1. **First run:** Run spec-review on a spec directory → confirm `docs/.cache/spec-review/` is created with manifest + summaries
2. **No-change re-run:** Run spec-review again without changing any files → confirm it loads from cache (no full file reads)
3. **Partial change:** Modify one spec file, re-run → confirm only that file is re-read and its summary updated
4. **New file:** Add a new spec file, re-run → confirm it's read and added to cache
5. **Delete file:** Remove a spec file, re-run → confirm it's removed from cache
6. **Force refresh:** Delete `docs/.cache/spec-review/`, re-run → confirm full read and cache rebuild
7. **Context savings:** Compare token usage between cached and uncached runs on a large spec corpus

### Design Decisions

- **Summaries are Markdown, not JSON** — inspectable, debuggable, and the LLM can read them naturally
- **Per-skill cache directories** — avoids conflicts between skills that might summarize the same file differently
- **SHA256 for hashing** — widely available via `shasum -a 256` on macOS/Linux, no dependencies needed
- **No code, only Markdown** — this is a skill-level mechanism described in reference docs, consistent with PMP's document-driven architecture
- **Cache is gitignored** — it's derived data that can be regenerated from source files
