# Discovery & System Reconstruction

Shared foundation for all spec-review sub-commands. Contains Phase 0 (Discovery) and Phase 1 (System Reconstruction), plus context management rules and prior review annotation handling.

When invoked by the orchestrator (`/pmp:spec-review`), these phases run once and results are available to all sub-commands. When a sub-command runs standalone, it executes these phases itself.

> **Reference:** Analysis cache details in [analysis-cache.md](../../pmp/references/analysis-cache.md)

---

## Phase 0: Discovery

1. **Map the specification corpus**
   - List all spec files in the provided location(s)
   - Identify the document hierarchy and relationships
   - Note any README or index files that describe reading order

2. **Classify documents**
   - **Primary specs**: Markdown/text documents defining the system (review targets)
   - **Reference material**: OpenAPI schemas, JSON schemas, ADRs, code comments (use for cross-validation, don't review independently)

3. **Collect prior review annotations**
   - Scan all primary spec files for GitHub blockquote alerts (`> [!WARNING]`, `> [!NOTE]`, `> [!CAUTION]`) that contain `**Reviewed`
   - Parse each annotation into: `{ date, finding_title, decision (acknowledged/skipped), reason, file, line }`
   - Build a **prior decisions registry** — a lookup table of previously reviewed findings
   - Report: `Found N prior review annotations across M files`
   - These annotations are produced by the `/pmp:discuss` skill and will be cross-referenced during analysis phases to avoid blindly re-raising resolved findings (see "Using Prior Review Annotations" below)

4. **Check analysis cache (MANDATORY)** — see [analysis-cache.md](../../pmp/references/analysis-cache.md). This step is not optional — always check the cache before reading files.
   - Check for `docs/.cache/spec-review/manifest.json`
   - If manifest exists: hash each spec file (`shasum -a 256`), compare to manifest
     - **Unchanged files** → load cached summaries instead of reading source
     - **Changed files** → read source, regenerate summary, update cache
     - **New files** → read source, generate summary, add to cache
     - **Deleted files** → remove from manifest and delete summary
   - If no manifest: proceed to step 5 (full read), then build cache afterward
   - Report cache status: `X cached, Y changed, Z new, W deleted`

5. **Read documents** (full read on cold cache, changed/new only on warm cache)
   - Read every uncached spec file before beginning analysis
   - Track key terms, states, field names, and identifiers as you read
   - Note cross-references between documents
   - After reading, produce a structured summary per file using the spec-review extraction template from [analysis-cache.md](../../pmp/references/analysis-cache.md) and write to `docs/.cache/spec-review/summaries/`
   - Write or update `docs/.cache/spec-review/manifest.json`

6. **Corpus Health Assessment**

   After reading all files, compute organization health signals to determine whether the corpus would benefit from reorganization via `/pmp:arc42`:

   | Signal | Metric | Threshold |
   |--------|--------|-----------|
   | **Concern scatter** | How many files share the same primary concern | >2 files per concern = scattered |
   | **Duplicate density** | % of sections with near-identical content in other files | >15% = high duplication |
   | **Cross-reference density** | Avg internal links per file | >5 = tangled dependencies |
   | **Orphan content** | Sections that don't fit the file's primary concern | >20% of sections = misplaced content |
   | **File count vs concern count** | Ratio of files to distinct concerns | >2:1 = over-fragmented |

   **If 2+ signals exceed thresholds**, include a `## Corpus Health Advisory` section in the report (see template) recommending `/pmp:arc42`. This is informational only — proceed with the full review regardless.

### Using Prior Review Annotations

During all subsequent analysis phases, before reporting a finding, check if it matches a prior annotation from the registry built in Phase 0 step 3. Match by title similarity or by referring to the same spec file section where an annotation exists.

- **`[!CAUTION]` or `[!WARNING]` match (Acknowledged):** Still include the finding in the report, but prefix it with `[Previously acknowledged YYYY-MM-DD: "reason"]`. Reduce its effective severity by one level (Critical stays Critical; Important becomes Minor; Minor is omitted unless circumstances have materially changed since the annotation date).
- **`[!NOTE]` match (Skipped):** Include the finding normally — skipped means "not decided", so it should get a fresh evaluation. Add a note: `[Previously skipped YYYY-MM-DD]`.
- **No match:** Report normally.

This ensures acknowledged findings appear with context rather than being blindly re-raised, while skipped findings get a fresh evaluation.

---

## Context Management

**All analysis phases run in the main controller context — do NOT spawn agents for analysis.**

Agents share no context with the main controller. Spawning agents for analysis phases forces each agent to re-read all files, multiplying token cost by the number of agents. Instead:

- **Phase 0 (Discovery):** Use parallel agents ONLY for reading files when the corpus is large (10+ files). Each agent reads a partition of files and returns the content. The main controller collects all file contents.
- **All analysis phases:** Run sequentially in the main controller. The controller already has all file contents from Phase 0 — no re-reading needed.
- **Remediation phase:** Naturally cross-references findings from all phases since everything is in one context.

### Large Corpus Fallback: Batched Phases

When the corpus is large enough to risk context overflow (estimated >30 files or >50K words of source), split analysis into batches:

1. **Batch 1:** Discovery + System Reconstruction + spec-architecture
2. **Batch 2:** spec-security
3. **Batch 3:** spec-operations + spec-implementability + Remediation

Between batches, produce a **checkpoint summary**:
- Key findings from completed phases (structured, ~500 words)
- The system model from Phase 1 (components, boundaries, state — ~300 words)
- List of files most relevant to the next batch

Each new batch starts by re-reading the source files + the checkpoint. This means files are re-read 2-3x total instead of N-agents x all-files. The checkpoint ensures cross-phase coherence.

**Batch boundaries are phase-aware:** never split a logically connected pair (e.g., threat modeling + attack simulation stay together).

---

## Phase 1: System Reconstruction

Build a clear model of the system. Extract and describe:

### 1. Components

List all components and classify them as:
- data plane
- control plane
- policy engines
- observability systems
- external dependencies

For each component identify:
- responsibility
- inputs
- outputs
- state
- failure modes

### 2. Request Lifecycle

Describe the **exact lifecycle of a request**, step-by-step from:

`client → ingress → proxy → policy → upstream → response`

Include: authentication, authorization, policy evaluation, request mutation, tool invocation, streaming, logging, auditing.

### 3. Trust Boundaries

Identify all security boundaries (e.g., internet → proxy, proxy → internal services, proxy → external APIs, proxy → policy engine, proxy → logging infrastructure).

Mark which components are trusted vs untrusted.

### 4. State Model

Identify all state in the system (caches, policy state, request context, connection pools, queues, session tokens, streaming buffers).

Determine whether state is: local, shared, replicated, or eventually consistent.
