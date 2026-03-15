# Plan: Add `/pmp:arc42` Sub-Skill

## Context

Specs and architecture docs grow organically over many revisions and reviews. Content gets scattered across files, duplicated, and disorganized. There's currently no PMP command to consolidate them. This skill reorganizes spec/architecture files into the **arc42** standard structure — one file per arc42 section, merging duplicates, flagging contradictions, and preserving review annotations.

### Arc42 Target Structure

The [arc42 template](https://arc42.org/overview) provides 12 standardized sections for architecture documentation. Each section becomes a **directory** with a `README.md` (section overview/index) and one file per concern/topic. Only sections with actual content are created.

| # | Section | Directory | Content Examples |
|---|---------|-----------|-----------------|
| 1 | Introduction & Goals | `01-introduction-and-goals/` | Requirements, stakeholders, quality goals |
| 2 | Constraints | `02-constraints/` | Technical, organizational, regulatory constraints |
| 3 | Context & Scope | `03-context-and-scope/` | System boundaries, external interfaces, actors |
| 4 | Solution Strategy | `04-solution-strategy/` | Key design decisions, technology choices, patterns |
| 5 | Building Block View | `05-building-block-view/` | Components, modules, data models, API contracts |
| 6 | Runtime View | `06-runtime-view/` | Request flows, sequences, state machines, pipelines |
| 7 | Deployment View | `07-deployment-view/` | Infrastructure, environments, CI/CD, containers |
| 8 | Crosscutting Concepts | `08-crosscutting-concepts/` | Security, error handling, logging, auth, i18n |
| 9 | Architecture Decisions | `09-architecture-decisions/` | ADRs, design rationale, rejected alternatives |
| 10 | Quality Requirements | `10-quality-requirements/` | Performance targets, SLAs, quality scenarios |
| 11 | Risks & Technical Debt | `11-risks-and-technical-debt/` | Known risks, tech debt, deferred items |
| 12 | Glossary | `12-glossary/` | Domain terms, abbreviations, definitions |

**Structure within each section directory:**
```
NN-section-name/
├── README.md              ← Section overview, subsection guidance per arc42 tips, index of files
├── <concern-a>.md         ← One file per concern/topic within this section
├── <concern-b>.md
└── ...
```

Content that doesn't fit any arc42 section goes into a `99-appendix/` directory (rare — arc42 covers nearly everything).

## Overview

Create a new standalone sub-skill (`/pmp:arc42`) that:
1. Reads all spec/architecture files in a user-provided directory
2. Classifies content into arc42 sections
3. Detects duplicates, overlaps, and contradictions
4. Proposes a reorganized arc42 file structure
5. Executes the reorganization after user approval
6. Verifies no content was lost

## Files to Create

### 1. `plugins/pmp/skills/arc42/SKILL.md`

Skill entry point with frontmatter. Follows the same pattern as `spec-review/SKILL.md`:
- **name:** `arc42`
- **description:** Trigger phrases: "organize specs", "arc42", "consolidate docs", "clean up specs", "reorganize architecture", "merge spec files", "deduplicate specs"
- **Workflow:** Read config → check analysis cache (`docs/.cache/arc42/`) → read `references/arc42.md` and follow it → present proposal → execute after approval → verify
- References shared resources: `config.md`, `analysis-cache.md`, `overview.md`

### 2. `plugins/pmp/skills/arc42/references/arc42.md`

Main algorithm file (the detailed methodology). Five phases:

**Phase 1 — Discovery:**
- Map the corpus (list files, classify as spec/reference/meta)
- Check analysis cache (mandatory — `docs/.cache/arc42/manifest.json`)
- Read uncached files, produce structured summaries with arc42 section tags, content fingerprints, cross-references, and annotation inventory

**Phase 2 — Analysis:**
- Read [arc42-guide.md](references/arc42-guide.md) for section definitions, content classification patterns, and per-section tips
- Classify every section into one of the 12 arc42 sections using the content classification guide in arc42-guide.md
- Build a section assignment table showing where each source section maps
- Detect duplicates (exact + semantic via content fingerprints)
- Detect contradictions (conflicting values, behaviors, terminology) — flag, never resolve
- Design target structure: one directory per arc42 section (`NN-section-name/`), only for sections with content. Within each directory, one file per concern/topic. Content that doesn't fit any section → `99-appendix/`
- Handle special content: preserve review annotations (`[!CAUTION]`, `[!WARNING]`, `[!NOTE]`), track internal cross-references, note YAML frontmatter

**Phase 3 — Proposal:**
- Present reorganization report using template (see asset below)
- Show: current inventory, proposed structure, content movement map, duplicates to merge, contradictions requiring user resolution, files to delete, files unchanged
- User options: Approve / Adjust / Abort
- Loop on Adjust until Approve or Abort

**Phase 4 — Execution:**
- Create arc42 section directories (`NN-section-name/`) for each section that has content
- Create `README.md` in each section directory with: section title, arc42 subsection guidance (from arc42-guide.md), and index of concern files within the directory
- Create one concern file per topic within each section directory, with provenance comments (`<!-- Consolidated from: ... on YYYY-MM-DD -->`)
- Structure content within each concern file following arc42 tips (e.g., Section 5 uses hierarchical levels with black/white box descriptions; Section 9 uses ADR format; Section 10 uses quality scenario format)
- Remove exact duplicates, merge semantic duplicates with `<!-- Merged from: ... -->` comments
- Preserve all review annotations attached to their content
- Insert contradiction markers for any unresolved contradictions
- Update all internal cross-references to new file/directory paths
- Create top-level `README.md` as master index linking to all section directories
- Delete fully-consumed source files (never delete reference material)

**Phase 5 — Verification:**
- Content coverage check (every source section appears in target)
- Annotation preservation check
- Cross-reference validity check
- Summary report with metrics (files processed/created/removed, sections moved, duplicates merged, annotations preserved)

**Constraints:**
- Corpus size limits: <30 files normal, 30-60 warn, >60 require focus area
- Never silently resolve contradictions
- Never delete reference material (schemas, ADRs, configs)
- Never skip proposal phase
- Reorganization is structural, not editorial (don't rewrite content)

### 3. `plugins/pmp/skills/arc42/references/arc42-guide.md`

Reference file containing arc42's section guidance — what belongs in each section, recommended form, and key tips. This is read during Phase 2 (Analysis) to correctly classify content AND during Phase 4 (Execution) to structure each output file according to arc42 best practices.

**Per-section guidance to include:**

| Section | Subsections / Form | Key Tips |
|---------|-------------------|----------|
| **1. Introduction & Goals** | 1.1 Requirements Overview (short text/table), 1.2 Quality Goals (top 3-5, table with scenarios), 1.3 Stakeholders (table: role, contact, expectations) | Keep compact (<1 page). Focus on essential tasks. Use scenarios for quality goals. |
| **2. Constraints** | Technical constraints, Organizational constraints, Conventions (coding, naming, versioning, docs) | Subdivide by category. Clarify consequences. Note which are negotiable. |
| **3. Context & Scope** | 3.1 Business Context (domain interfaces, I/O), 3.2 Technical Context (channels, protocols, hardware) | Show as diagram + table. Restrict to overview. Show data flows not just dependencies. |
| **4. Solution Strategy** | Table mapping quality goals → solution approaches → section references | Keep compact (keywords). Always justify choices. Reference concepts/views for support. |
| **5. Building Block View** | Level 1 (whitebox of overall system), Level 2 (selected block refinements), Level 3+ (as needed). Black box = responsibility + interfaces. White box = internal structure. | Always describe Level 1. Use common structures. Organize hierarchically. Prefer relevance over completeness. |
| **6. Runtime View** | Named scenarios showing building block interactions. Use: step lists, activity/sequence diagrams, BPMN, state machines. | Document only representative scenarios. Use schematic, not detailed. Map to existing building blocks. |
| **7. Deployment View** | Infrastructure topology (nodes, channels), software-to-hardware mapping, environment descriptions | Document multiple environments. Use UML deployment diagrams or tables for mapping. Explain nodes. |
| **8. Crosscutting Concepts** | Concept papers per topic (security, logging, error handling, persistence, UI, etc.) | Document only critical topics. Support with code examples. Link between building blocks and concepts. Prioritize decisions over general concepts. |
| **9. Architecture Decisions** | ADRs (Title, Context, Decision, Status, Consequences) or decision tables | Focus on architecturally relevant decisions. Include criteria and rationale. Document rejected alternatives. Add timestamps. |
| **10. Quality Requirements** | 10.1 Overview table (categories from ISO 25010), 10.2 Quality Scenarios (source, stimulus, environment, response, measure) | Include usage, change, and failure scenarios. Use as verification checklist. |
| **11. Risks & Technical Debt** | Prioritized list with mitigation strategies | Engage different stakeholders. Examine external interfaces. Review data handling. |
| **12. Glossary** | Table: Term, Definition (optional: Translation columns) | Treat seriously. Keep concise — exclude trivial terms. Assign maintenance ownership. |

**Content classification guide** (used during Phase 2 to map source content → arc42 sections):

| Source Content Pattern | Maps To |
|----------------------|---------|
| Business goals, feature lists, user stories, requirements | Section 1 |
| Technology mandates, regulatory requirements, coding standards | Section 2 |
| System boundaries, external APIs, actors, neighboring systems | Section 3 |
| "Why we chose X", technology rationale, approach justification | Section 4 |
| Component descriptions, module APIs, data models, class hierarchies | Section 5 |
| Request flows, sequence diagrams, state machines, use case walkthroughs | Section 6 |
| Infrastructure, Docker/K8s configs, CI/CD, environment descriptions | Section 7 |
| Security policies, auth patterns, error handling, logging, i18n | Section 8 |
| ADRs, "we decided to...", design trade-offs, rejected alternatives | Section 9 |
| SLAs, performance targets, quality attributes, test criteria | Section 10 |
| Known issues, tech debt backlog, risk registers, deferred work | Section 11 |
| Term definitions, abbreviations, domain vocabulary | Section 12 |

### 4. `plugins/pmp/skills/arc42/assets/reorganization-report.md` (same as before, renumbered)

Template for the proposal report, containing tables for:
- Current file inventory (file, word count, arc42 section mapping)
- Arc42 section inventory (which sections will be created, section count, source files)
- Proposed file structure (target arc42 file, sources consolidated into it)
- Content movement map per source file (section → target arc42 file, action)
- Duplicates to merge (content summary, locations, keep-from recommendation)
- Contradictions requiring resolution (both statements, sources, severity)
- Files to delete / files unchanged
- Review annotations inventory (type, count, source files)

## Files to Modify

### 5. `plugins/pmp/skills/spec-review/references/spec-review.md`

Add a **Corpus Health Assessment** step to Phase 0 (Discovery), after step 5 (Read documents). This step evaluates whether the spec corpus would benefit from reorganization before proceeding with the full review.

**New step 6 — Corpus Health Assessment:**

After reading all files, compute organization health signals:

| Signal | Metric | Threshold |
|--------|--------|-----------|
| **Concern scatter** | How many files share the same primary concern | >2 files per concern = scattered |
| **Duplicate density** | % of sections with near-identical content in other files | >15% = high duplication |
| **Cross-reference density** | Avg internal links per file | >5 = tangled dependencies |
| **Orphan content** | Sections that don't fit the file's primary concern | >20% of sections = misplaced content |
| **File count vs concern count** | Ratio of files to distinct concerns | >2:1 = over-fragmented |

**If 2+ signals exceed thresholds**, add a `Corpus Health` section to the report (before the analysis phases) with:

```markdown
## Corpus Health Advisory

**Recommendation:** Run `/pmp:arc42` before or after addressing review findings.

| Signal | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Concern scatter | [N] files share concerns | >2 | [WARN/OK] |
| Duplicate density | [N]% | >15% | [WARN/OK] |
| ... | | | |

**Rationale:** [1-2 sentences explaining why reorganization would improve the corpus]
```

This advisory is informational — spec-review still proceeds with the full analysis regardless. The user can choose to run `/pmp:arc42` before or after acting on the review findings.

**Also modify the report template** (`plugins/pmp/skills/spec-review/assets/spec-review-output.md`):

Add an optional `## Corpus Health Advisory` section between Executive Summary and System Model, with a comment: `<!-- OPTIONAL: Include when corpus health signals exceed thresholds. Delete if corpus is well-organized. -->`

### 6. `plugins/pmp/skills/pmp/SKILL.md`

- Add to **Sub-Skills** table: `| /pmp:arc42 | Consolidate scattered spec files into arc42 structure |`
- Add to **Routing** table: `| "organize specs", "arc42", "consolidate docs", "clean up specs", "reorganize architecture", "merge spec files" | Arc42 | arc42.md |`
- Add to **Lifecycle Flow** section: note that Arc42 is standalone (like Spec Review)
- Bump version from `1.7.1` to `1.7.4` (matching the `1.7.3` commit already made)

### 7. `plugins/pmp/skills/pmp/references/overview.md`

- Add `Arc42` state to the **Detailed State Diagram** (standalone, like SpecReview)
- Add to **Five Entry Points** standalone modes table: `| "Organize specs" / "Arc42" | Arc42 — reorganize spec files into arc42 structure |`
- Add to **File Structure** tree: the `arc42/` directory with its files
- Add to **Assets** table: `| reorganization-report.md | Reorganization proposal template | arc42.md |`
- Add **1.7.4** changelog entry

### 8. `plugins/pmp/skills/pmp/config.md`

- Add to **Stage Announcements** table: `| Arc42 | Reorganizing specs into arc42 structure... |`

### 9. `plugins/pmp/.claude-plugin/plugin.json`

- Bump version to `1.7.4`
- Add "arc42" to keywords if applicable

## Integration

- **Standalone workflow** — like spec-review, does not feed into plan generation or execution
- **Spec-review recommends arc42:** During Discovery, spec-review computes corpus health signals (concern scatter, duplicate density, cross-reference density, orphan content, fragmentation ratio). If 2+ signals exceed thresholds, the review report includes a "Corpus Health Advisory" recommending `/pmp:arc42`
- **Natural pairing with discuss:** if discuss reveals disorganized specs, user can run `/pmp:arc42`
- **Uses existing infrastructure:** analysis cache, TodoWrite progress tracking, AskQuestion transitions

## Verification

1. Invoke `/pmp:arc42` with a test spec directory containing:
   - Multiple files with overlapping content
   - Files with review annotations
   - Files with cross-references to each other
   - A contradiction between two files
2. Verify the proposal phase shows all duplicates, contradictions, and proposed structure
3. Approve and verify execution produces clean arc42-structured output (numbered directories with README + concern files)
4. Verify all review annotations are preserved
5. Verify cross-references are updated
6. Verify the verification phase reports no content loss
7. Test the router: say "organize my specs" or "arc42" to `/pmp` and verify it routes to `/pmp:arc42`
8. Run `/pmp:spec-review` on a disorganized spec corpus and verify the report includes a "Corpus Health Advisory" recommending `/pmp:arc42`
