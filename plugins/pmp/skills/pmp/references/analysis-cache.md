# Analysis Cache

Reusable per-file digest cache for skills that read spec/architecture files upfront. Avoids re-reading unchanged files on subsequent runs by storing structured summaries alongside content hashes.

## When to Use

Any skill that reads a corpus of files before analysis (spec-review, plan, review) can opt into this cache. The cache is especially valuable when:
- The spec corpus is large (10+ files)
- The skill is run multiple times against the same specs
- Context window budget is tight

## Cache Structure

```
docs/.cache/<skill-name>/
├── manifest.json
└── summaries/
    ├── <sanitized-filename-1>.md
    ├── <sanitized-filename-2>.md
    └── ...
```

**Sanitized filenames:** Replace `/` with `--`, drop file extension, append `.md`.
Example: `specs/auth/oauth.md` → `specs--auth--oauth.md`

## Manifest Schema

```json
{
  "version": 1,
  "skill": "<skill-name>",
  "source_dir": "<path provided by user>",
  "created_at": "<ISO 8601 UTC>",
  "updated_at": "<ISO 8601 UTC>",
  "files": {
    "<relative-file-path>": {
      "sha256": "<hex digest>",
      "summary_file": "summaries/<sanitized-name>.md",
      "extracted_at": "<ISO 8601 UTC>",
      "word_count": 3200
    }
  }
}
```

## Summary File Format

Each summary has YAML frontmatter linking it to its source, followed by structured extraction sections. The sections vary by skill — each skill defines its own extraction template.

```markdown
---
source: <relative-file-path>
sha256: <hex digest>
extracted_at: <ISO 8601 UTC>
---

[Structured extraction sections — skill-specific]
```

## Cache Check Algorithm

Insert this step into the skill's discovery/input phase, **after** listing and classifying files but **before** reading them:

### Step 1: Check for Manifest

```
manifest_path = docs/.cache/<skill>/manifest.json
```

If `manifest_path` does not exist → **cold cache** (skip to Step 4).

### Step 2: Hash and Compare

For each file in the current spec corpus:
1. Compute SHA256: `shasum -a 256 <file>`
2. Look up file in manifest
3. Classify:
   - **Unchanged:** SHA256 matches manifest → file is cached
   - **Changed:** SHA256 differs → needs re-read and re-extraction
   - **New:** File not in manifest → needs read and extraction
   - **Deleted:** File in manifest but not on disk → remove from cache

### Step 3: Load Context (Warm Cache)

- Load summaries for **unchanged** files (read from `docs/.cache/<skill>/summaries/`)
- Read **changed** and **new** source files in full
- After reading changed/new files, generate summaries and update cache
- Remove entries for deleted files (delete summary file + manifest entry)
- Update `manifest.json` with new hashes and timestamp

Report to user:
```
Cache status: X files cached, Y changed (re-read), Z new (read), W deleted (removed)
```

### Step 4: Cold Cache (First Run)

- Read all files normally (existing behavior)
- After reading each file, produce a structured summary
- Write summaries to `docs/.cache/<skill>/summaries/`
- Write `manifest.json` with all file hashes
- Continue with analysis using full context

Report to user:
```
No cache found. Reading all files and building cache for next run.
```

## Cache Invalidation

- **Automatic:** Content hash mismatch triggers re-read of that file only
- **Manual:** Delete `docs/.cache/<skill>/` to force a complete refresh
- **No time-based expiry** — content hashing is the sole invalidation signal

## Adopting in a New Skill

To add caching to a skill:

1. **Define an extraction template** — what structured sections should each summary contain? This depends on what the skill needs from its input files. See the spec-review extraction template below as an example.

2. **Add the cache check** — insert the cache check algorithm (above) into the skill's discovery/input reading phase, between file classification and file reading.

3. **Use the standard directory** — store in `docs/.cache/<skill-name>/` per [config.md](../pmp/config.md) File Paths.

## Spec-Review Extraction Template

When spec-review generates a summary for a cached file, extract these sections:

```markdown
## Components
- [component list with classification: data plane, control plane, policy, observability, external]
- [for each: responsibility, inputs, outputs, state, failure modes]

## Key Decisions
- [architectural decisions, trade-offs, constraints stated in this file]

## External Dependencies
- [APIs, services, databases, external systems referenced]

## State & Data Flow
- [state management patterns, data flow, request lifecycle details]

## Cross-References
- [references to other spec files, shared concepts, linked documents]

## Security Boundaries
- [trust boundaries, auth mechanisms, access control defined here]

## Terms & Identifiers
- [key terms, field names, identifiers, enums defined in this file]
```

## Design Decisions

- **Summaries are Markdown** — inspectable, debuggable, LLM-readable
- **Per-skill directories** — avoids conflicts when skills extract different things from the same file
- **SHA256 hashing** — available via `shasum -a 256` on macOS/Linux, no dependencies
- **No code** — this is a skill-level mechanism described in Markdown, consistent with PMP's document-driven architecture
- **Gitignored** — `docs/.cache/` is derived data, regenerated from source files
