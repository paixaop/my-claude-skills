# PMP + GitHub Planning

An AI-agent-driven planning and execution system. Goes from idea to merged PR with full E2E test coverage and GitHub Issue tracking.

## How It Works

### The Lifecycle

```mermaid
graph LR
    A[Idea / Spec / Issues] --> B[Generate Plan]
    B --> C[Plan Review]
    C --> D{GitHub Issues?}
    D -->|Create new| E[GitHub Planning]
    D -->|Already exist| F[Execute]
    D -->|Skip| F
    E --> F
    F --> G[PR with 'Closes #N']
    G -->|Merge| H[Issues auto-closed]
```

Every transition between stages requires user confirmation. The agent never auto-advances.

## Detailed State Diagram

```mermaid
stateDiagram-v2
    [*] --> Brainstorm : idea / feature request
    [*] --> GeneratePlan : spec / roadmap provided
    [*] --> FetchIssues : GitHub issues / epic provided
    [*] --> PlanReview : existing plan file
    [*] --> SpecReview : "review specs" / architecture review
    [*] --> GitHubPlanning : "create issues" / "make epic"
    [*] --> SyncIssues : "update issues" / "sync issues"
    [*] --> TestOnly : "run tests" / "re-test"

    state FetchIssues {
        [*] --> IdentifyEpic : issue URL / number
        IdentifyEpic --> FetchSubIssues : gh api graphql
        FetchSubIssues --> ParseIssueBodies
        ParseIssueBodies --> BuildRoadmap : normalize to feature specs
        BuildRoadmap --> PreFillIssueTable : issues already exist
        PreFillIssueTable --> [*]
    }

    FetchIssues --> GeneratePlan : issues fetched

    state Brainstorm {
        [*] --> Explore : read context
        Explore --> AskClarify
        AskClarify --> ProposeApproaches
        ProposeApproaches --> PresentDesign
        PresentDesign --> Refine
        Refine --> AskClarify : user wants changes
        Refine --> DesignDoc : user approves
        DesignDoc --> [*]
    }

    Brainstorm --> GeneratePlan : user confirms

    state GeneratePlan {
        [*] --> ReadRoadmap
        ReadRoadmap --> AnalyzeCodebase
        AnalyzeCodebase --> ClarifyRequirements
        ClarifyRequirements --> WritePlan
        WritePlan --> VerifyCoverage
        VerifyCoverage --> SavePlan
        SavePlan --> [*]
    }

    GeneratePlan --> PlanReview : user confirms

    state PlanReview {
        [*] --> ReadPlan
        ReadPlan --> ReadCode
        ReadCode --> SecurityGate
        SecurityGate --> ScoreChecklist
        ScoreChecklist --> ProduceReview

        ProduceReview --> UpdatePlan : user picks "update"
        ProduceReview --> Discuss : user picks "discuss"
        ProduceReview --> Approved : user picks "proceed"

        UpdatePlan --> ReadPlan : re-review
        Discuss --> ProduceReview : re-ask
        Approved --> [*]
    }

    PlanReview --> GitHubPlanning : user confirms
    PlanReview --> Execute : user declines issues

    state SpecReview {
        [*] --> MapCorpus
        MapCorpus --> ClassifyDocs
        ClassifyDocs --> ReadAllSpecs
        ReadAllSpecs --> ReconstructSystem
        ReconstructSystem --> DeepAnalysis
        DeepAnalysis --> ProduceReport

        ProduceReport --> DiscussFindings : user picks "discuss"
        ProduceReport --> ReviewDone : user picks "done"

        DiscussFindings --> ProduceReport : re-ask
        ReviewDone --> [*]
    }

    SpecReview --> [*] : report delivered

    state GitHubPlanning {
        [*] --> DetermineTier
        DetermineTier --> PreFlight
        PreFlight --> CreateLabels
        CreateLabels --> CreateIssues
        CreateIssues --> RecordMapping : write to plan file
        RecordMapping --> [*]
    }

    GitHubPlanning --> Execute : user confirms

    state Execute {
        [*] --> SetupInfra : first run
        SetupInfra --> FeatureLoop

        state FeatureLoop {
            [*] --> TaskA_Impl : Task A
            TaskA_Impl --> RunUnitTests
            RunUnitTests --> UnitFixLoop : tests fail
            RunUnitTests --> CommitImpl : all pass
            UnitFixLoop --> RunUnitTests : fix applied
            UnitFixLoop --> Blocked : ceiling reached
            CommitImpl --> TaskB_E2E : Task B
            TaskB_E2E --> RunE2ETests
            RunE2ETests --> E2EFixLoop : tests fail
            RunE2ETests --> CommitE2E : all pass
            E2EFixLoop --> RunE2ETests : fix applied
            E2EFixLoop --> Blocked : ceiling reached
            CommitE2E --> CommentOnIssue : if GH issues exist
            CommentOnIssue --> [*]
            CommitE2E --> [*] : no GH issues
        }

        FeatureLoop --> FeatureLoop : next feature
        FeatureLoop --> FinalRun : all features done
        FinalRun --> CreatePR
        CreatePR --> PRClosesIssues : GH issues (Closes #N in body)
        PRClosesIssues --> UpdateFrontmatter
        CreatePR --> UpdateFrontmatter : no GH issues
        UpdateFrontmatter --> ArchivePlan : move to docs/plans/implemented/
        ArchivePlan --> [*]
    }

    state SyncIssues {
        [*] --> ReadPlanForSync
        ReadPlanForSync --> FetchCurrentIssues
        FetchCurrentIssues --> DiffBodies
        DiffBodies --> PresentDiff
        PresentDiff --> ApplySync : user approves
        ApplySync --> UpdateEpicBody
        UpdateEpicBody --> UpdatePlanTable
        UpdatePlanTable --> [*]
    }

    SyncIssues --> [*] : synced

    TestOnly --> [*] : results reported

    Execute --> [*] : done
```

### Five Entry Points

| You say... | What happens |
|------------|--------------|
| "I have an idea for..." | **Workflow 1** -- Brainstorm → Plan → Plan Review → Execute |
| "Here's my roadmap/spec" | **Workflow 2** -- Plan → Plan Review → Execute |
| "Plan from epic #41" | **Workflow 4** -- Fetch Issues → Plan → Plan Review → Execute |
| "Review this plan" | **Workflow 3** -- Plan Review → Execute |
| "Review specs" / "Architecture review" | **Workflow 5** -- Architecture & Spec Review (standalone report) |

Plus standalone modes:

| You say... | What happens |
|------------|--------------|
| "Create issues for this plan" | GitHub Planning only (plan already exists) |
| "Update issues" / "Sync issues" | Sync Issues — diff plan against live issues, push changes |
| "Run E2E tests" | Test Only mode (no implementation) |

## The GitHub Integration

### Direction 1: Plan → Issues

After the agent generates and reviews a plan, it offers to publish it as GitHub Issues.

```
Plan approved
  → Agent determines complexity tier:
      SIMPLE (1-3 tasks)  → Single issue with checklist
      STANDARD (4-10)     → Epic + native sub-issues + milestone
      COMPLEX (10+)       → Epic + sub-issues + Projects v2 board
  → Creates issues with labels, verification criteria, and source context
  → Writes issue mapping table into the plan file
```

### Direction 2: Issues → Plan

A PM creates the epic and sub-issues in GitHub (manually or via the github-planning reference). Then tells the agent to plan from them.

```
PM creates Epic #41 with sub-issues #42, #43, #44
  → Agent fetches via `gh api graphql`
  → Parses issue bodies into feature specs
  → Generates full implementation plan with E2E tests
  → Pre-fills the GitHub Issues table (no need to create issues — they exist)
  → Normal review → execute flow
```

### How Issues Get Closed

Issues are **never manually closed**. The PR does it.

```
During execution:
  Feature committed → agent comments on issue with commit SHA
  ...

After all features pass:
  Agent creates PR with body:
    Closes #42
    Closes #43
    Closes #44
    Closes #41  ← the epic

PR merges → GitHub auto-closes all issues
```

## File Structure

```
pmp/
├── SKILL.md                            # Main skill — workflows, lifecycle, rules
├── config.md                           # Central constants — paths, thresholds, labels, announcements
├── references/
│   ├── brainstorm.md                   # Collaborative design stage
│   ├── generate-plans.md               # Plan generation (+ GitHub Issues Mode)
│   ├── review.md                       # Plan review — skeptical senior engineer
│   ├── spec-review.md                  # Architecture & spec review — 15-phase deep analysis
│   ├── execute-loop.md                 # Code-test-fix loop with E2E + agent teams
│   ├── github-planning.md              # Issue/Epic/Project creation
│   ├── sync-issues.md                  # Sync plan changes to existing issues
│   ├── testing-approaches.md           # Per-project-type E2E guidance
│   ├── security-analysis.md            # STRIDE + attack tree analysis
│   ├── implementer-prompt.md           # Agent team: implementer
│   ├── code-quality-reviewer-prompt.md # Agent team: code reviewer
│   └── spec-reviewer-prompt.md         # Agent team: spec reviewer
└── assets/
    ├── plan.md                         # Full implementation plan structure
    ├── design-doc.md                   # Design document from brainstorm
    ├── feature.md                      # Feature spec with ACs and E2E tests
    ├── task.md                         # TDD task with steps
    ├── review-output.md                # Plan review verdict and findings
    ├── spec-review-output.md           # Architecture & spec review report
    ├── issue-simple.md                 # SIMPLE tier: single issue body
    ├── issue-epic.md                   # STANDARD/COMPLEX: epic body
    ├── issue-sub-issue.md              # Sub-issue body
    ├── pr-body.md                      # Pull request body
    ├── e2e-test-spec.md                # Agent-driven test spec format
    ├── security-analysis-output.md     # Security analysis report
    ├── github-issues-table.md          # Feature→Issue mapping table
    ├── phase-exit-criteria.md          # Phase gate checklist
    ├── yaml-feature-form.yml           # GitHub Issue form: feature
    ├── yaml-bug-form.yml               # GitHub Issue form: bug
    └── yaml-epic-form.yml              # GitHub Issue form: epic
```

## Assets

Reusable templates for all artifacts. Reference files use these templates — read them when creating the corresponding artifact.

| Template | Purpose | Used By |
|----------|---------|---------|
| [plan.md](../assets/plan.md) | Full implementation plan structure | generate-plans.md |
| [design-doc.md](../assets/design-doc.md) | Design document from brainstorm | brainstorm.md |
| [feature.md](../assets/feature.md) | Feature spec with ACs and E2E tests | generate-plans.md |
| [task.md](../assets/task.md) | TDD task with steps | reference template |
| [review-output.md](../assets/review-output.md) | Plan review verdict and findings | review.md |
| [spec-review-output.md](../assets/spec-review-output.md) | Architecture & spec review report | spec-review.md |
| [issue-simple.md](../assets/issue-simple.md) | SIMPLE tier: single issue body | github-planning.md |
| [issue-epic.md](../assets/issue-epic.md) | STANDARD/COMPLEX tier: epic body | github-planning.md |
| [issue-sub-issue.md](../assets/issue-sub-issue.md) | Feature issue body (sub-issue of epic) | github-planning.md |
| [issue-task.md](../assets/issue-task.md) | Task issue body (sub-issue of feature) | github-planning.md |
| [pr-body.md](../assets/pr-body.md) | Pull request body | execute-loop.md |
| [e2e-test-spec.md](../assets/e2e-test-spec.md) | Agent-driven test spec format | execute-loop.md |
| [security-analysis-output.md](../assets/security-analysis-output.md) | Security analysis report | security-analysis.md |
| [github-issues-table.md](../assets/github-issues-table.md) | Feature→Issue mapping table | generate-plans.md, github-planning.md |
| [phase-exit-criteria.md](../assets/phase-exit-criteria.md) | Phase gate checklist | reference template |
| [yaml-feature-form.yml](../assets/yaml-feature-form.yml) | GitHub Issue form: feature | github-planning.md |
| [yaml-bug-form.yml](../assets/yaml-bug-form.yml) | GitHub Issue form: bug | github-planning.md |
| [yaml-epic-form.yml](../assets/yaml-epic-form.yml) | GitHub Issue form: epic | github-planning.md |

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
