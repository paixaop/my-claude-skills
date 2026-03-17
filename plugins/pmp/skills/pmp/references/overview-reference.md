# PMP Overview — Reference

> Part 2 of the overview. See [overview.md](overview.md) for the lifecycle, state diagram, entry points, GitHub integration, and file structure.

---

## Assets

Reusable templates for all artifacts. Reference files use these templates — read them when creating the corresponding artifact.

| Template | Purpose | Used By |
|----------|---------|---------|
| [plan.md](../../plan/assets/plan.md) | Full implementation plan structure | generate-plans.md |
| [design-doc.md](../../brainstorm/assets/design-doc.md) | Design document from brainstorm | brainstorm.md |
| [feature.md](../../plan/assets/feature.md) | Feature spec with ACs and E2E tests | generate-plans.md |
| [task.md](../assets/task.md) | TDD task with steps | reference template |
| [review-output.md](../../review/assets/review-output.md) | Plan review verdict and findings | review.md |
| [spec-review-output.md](../../spec-review/assets/spec-review-output.md) | Architecture & spec review report | spec-review.md |
| [issue-simple.md](../../github/assets/issue-simple.md) | SIMPLE tier: single issue body | github-planning.md |
| [issue-epic.md](../../github/assets/issue-epic.md) | STANDARD/COMPLEX tier: epic body | github-planning.md |
| [issue-sub-issue.md](../../github/assets/issue-sub-issue.md) | Feature issue body (sub-issue of epic) | github-planning.md |
| [issue-task.md](../../github/assets/issue-task.md) | Task issue body (sub-issue of feature) | github-planning.md |
| [pr-body.md](../../execute/assets/pr-body.md) | Pull request body | execute-loop.md |
| [e2e-test-spec.md](../../execute/assets/e2e-test-spec.md) | Agent-driven test spec format | execute-loop.md |
| [security-analysis-output.md](../../review/assets/security-analysis-output.md) | Security analysis report | security-analysis.md |
| [github-issues-table.md](../assets/github-issues-table.md) | Feature→Issue mapping table | generate-plans.md, github-planning.md |
| [phase-exit-criteria.md](../assets/phase-exit-criteria.md) | Phase gate checklist | reference template |
| [yaml-feature-form.yml](../../github/assets/yaml-feature-form.yml) | GitHub Issue form: feature | github-planning.md |
| [yaml-bug-form.yml](../../github/assets/yaml-bug-form.yml) | GitHub Issue form: bug | github-planning.md |
| [yaml-epic-form.yml](../../github/assets/yaml-epic-form.yml) | GitHub Issue form: epic | github-planning.md |
| [changelog-output.md](../../changelog/assets/changelog-output.md) | Release notes template | changelog.md |
| [reorganization-report.md](../../arc42/assets/reorganization-report.md) | Arc42 reorganization proposal template | arc42.md |

## Plan File Anatomy

Plans live in `docs/plans/` and get archived to `docs/plans/implemented/` after execution.

```markdown
# Auth System Implementation Plan

**Goal:** Add JWT-based authentication
**Source:** GitHub Epic #41              ← where the requirements came from
**Project Type:** REST/HTTP API
**Stack:** Go, PostgreSQL, chi router
**Integration Branch:** develop
**CI Command:** make ci

## GitHub Issues                        ← maps features to issues
| Feature | Issue | Status |
|---------|-------|--------|
| Feature 1: User registration | #42 | Open |
| Feature 2: Login endpoint | #43 | Open |
| Epic | #41 | Open |

## E2E Test Infrastructure              ← how tests run
...

## Feature 1: User Registration         ← behavioral spec (no code)
**Behavior:** ...
**Affected Files:** ...

### Acceptance Criteria
#### AC-1.1: Valid registration
**E2E Test:** ...                       ← test spec lives right under the AC
#### AC-1.S1: SQL injection prevention
**E2E Test:** ...                       ← security is a testable AC, not a vague guideline
```

## Key Principles

- **Plans describe WHAT, not HOW** — behavioral specs, no code snippets
- **Every AC has an E2E test** — traceability is structural, not cross-referenced
- **Security in every plan** — input validation, auth, injection risks as testable ACs
- **Deferred commits** — nothing is committed until all E2E tests pass
- **Evidence before assertions** — "should work" is not evidence, run the command
- **User controls transitions** — agent asks before every stage change
- **Issues close via PR** — never manually, so rejected PRs leave issues open
- **Templates for consistency** — all artifacts use shared templates from `assets/`

## Prerequisites

- `gh` CLI authenticated (`gh auth login`)
- Git repository with GitHub remote
- For Projects v2: GraphQL API access

## Quick Start

Tell the agent what you want:

```
"I want to add user authentication to my API"      → starts brainstorming
"Here's my roadmap: [paste/file]"                   → generates plan directly
"Plan from epic #41"                                → fetches issues, generates plan
"Create issues for docs/plans/2026-02-24-auth.md"   → publishes plan as issues
"Execute docs/plans/2026-02-24-auth.md"             → implements with E2E tests
```

The agent handles the rest — detecting your project type, choosing test frameworks, creating branches, and wiring up GitHub Issues.

## Changelog

### 1.7.4

- **`/pmp:arc42` sub-skill:** Reorganize spec/architecture files into the arc42 standard structure. Reads all files, classifies content into arc42's 12 sections, detects duplicates and contradictions, proposes a directory structure (one directory per section, one file per concern), and executes after user approval. Includes arc42-guide.md with per-section tips from the official arc42 documentation. Preserves all review annotations, adds provenance comments, and produces a verification report. Uses analysis cache for efficient re-runs.
- **Spec-review corpus health assessment:** Phase 0 (Discovery) now computes organization health signals (concern scatter, duplicate density, cross-reference density, orphan content, fragmentation ratio). If 2+ signals exceed thresholds, the review report includes a "Corpus Health Advisory" recommending `/pmp:arc42`.

### 1.7.1

- **Plan decomposition:** Auto-phasing for plans with 5+ features groups features into dependency-ordered phases with entry/exit criteria. New `/pmp:decompose` sub-skill for standalone phasing of existing plans. Execute loop uses phase boundaries as batch boundaries when present.
- **Structured approach comparison:** Brainstorm now produces a comparison matrix (4-6 dimensions) when proposing approaches, included in the design doc for traceability into plan generation.
- **Review re-run:** When re-reviewing a previously reviewed plan, the review skill loads the prior review, diffs the plan, and focuses on changed/new content. Produces a "Previous Finding Resolution" table tracking which findings were addressed. Security gate always re-runs fully.
- **`/pmp:changelog` sub-skill:** Generates user-facing release notes from completed plans by cross-referencing plan features against git commit history. Supports input via plan file path or PR number. Execute loop now offers changelog generation after archiving.

### 1.7.0

- **Analysis cache:** Per-file digest cache for skills that read spec files upfront. Stores structured summaries with SHA256 change detection — unchanged files load from cache on re-run. Any skill can opt in via `docs/.cache/<skill>/`. See `references/analysis-cache.md`.
- **`/pmp:discuss` sub-skill:** Structured walkthrough of review findings (plan review or spec-review). Iterates findings from most critical to least, explains each issue, proposes solutions, asks the user what to do (Acknowledge/Fix/Skip/Defer). Collects fixes into a plan and hands off to execute.
- **Spec-review caching:** Phase 0 (Discovery) now checks for cached summaries before reading files. Only changed/new files are re-read on subsequent runs.

### 1.6.0

- **Sub-skill architecture:** Split PMP into focused sub-skills (`pmp:brainstorm`, `pmp:plan`, `pmp:review`, `pmp:execute`, `pmp:spec-review`, `pmp:github`). Each loads only the context it needs. Root `pmp` skill becomes a router for ambiguous requests.
- **Improved descriptions:** Each sub-skill has a focused, trigger-optimized description instead of one monolithic description covering all use cases.
- **SKILL.md further compressed:** Root skill is now 72 lines (down from 530 in v1.4). Sub-skills average 28 lines each.

### 1.5.0

- **Two-task execution model:** Each feature now produces two separate commits — Task A (implementation + unit tests) and Task B (E2E tests). Halves per-task cognitive load.
- **3-level GitHub mapping:** Plan → Epic, Feature → Issue, Task → Sub-issue. Clean 1:1 mapping with consistent terminology throughout.
- **Task issue template:** New `assets/issue-task.md` for task-level sub-issues under feature issues.
- **Feature template update:** `assets/feature.md` now includes explicit Tasks table with Task A/B and issue references.
- **Terminology cleanup:** Complexity tiers use "features" not "tasks". Labels use `type:feature` for feature issues, `type:task` for task sub-issues.
- **Updated commit conventions:** `feat(<scope>): <feature>` for implementation, `test(<scope>): e2e tests for <feature>` for E2E.
- **Implementer scope reduction:** Implementer agents no longer write E2E tests — only unit/integration tests.
- **Structural restructure:** SKILL.md compressed to routing layer (~120 lines). Operational content moved to reference files for progressive disclosure.

### 1.4.0

- **Multi-session resume:** Added protocol for resuming execution after session breaks (execute-loop.md)
- **Two-stage review:** Consolidated subagent-driven review (spec compliance + code quality) into execute-loop.md from removed execute.md
- **Monorepo support:** Added detection and scoping for workspace-based monorepos
- **E2E opt-out:** Added guidance for projects where E2E tests don't apply (ask if in doubt)
- **Dependency management:** Added supply chain guidance to Project Rules
- **Plan diff summary:** Extend Mode now presents a structured change summary before saving
- **Security analysis persistence:** Reports saved to `docs/reviews/` alongside architecture reviews
- **Directory bootstrapping:** `generate-plans.md` creates `docs/plans/` if it doesn't exist
- **Removed:** `write.md` and `execute.md` (consolidated into `generate-plans.md` and `execute-loop.md`)
- **Fixed:** `overview.md` wrong archive path (`docs/implemented/` → `docs/plans/implemented/`)
- **Fixed:** `overview.md` file tree indentation
- **Fixed:** `github-planning.md` contextd pre-flight no longer fails when contextd is unavailable
