# PMP Skill Restructuring — Progressive Disclosure

## Context

The PMP skill's SKILL.md is ~530 lines, violating the skill writing guideline of <500 lines (ideally much less). It inlines detailed operational content that belongs in reference files — E2E project detection tables, GitHub planning procedures, parallel agent rules, and more. Much of this duplicates content already in `config.md`, `overview.md`, and `execute-loop.md`. The result: every time the skill triggers, ~530 lines load into context even though only the routing logic matters at that point.

**Goal:** Shrink SKILL.md to ~120 lines by making it a pure router + guardrails. Move operational details to the reference files that already own those workflows.

## Changes

### 1. Rewrite SKILL.md (~530 → ~120 lines)

**File:** `plugins/pmp/skills/pmp/SKILL.md`

New structure:

```
Frontmatter (5 lines) — unchanged
Title + one-liner (4 lines)
Critical rule: always ask before transitioning (3 lines)
Entry Points & Workflows table (30 lines) — compact routing table
  - Merges current "Entry Point Detection" table + workflow descriptions
  - Each row: signal → workflow name → reference files in order
  - No mermaid diagram (moves to overview.md)
Compact workflow notes (15 lines) — one paragraph per workflow, only what's unique
  - GitHub Planning offer after review
  - Workflow 4 skips GitHub Planning (issues already exist)
  - Workflow 5 is standalone
Project Rules (15 lines) — compressed bullet list, essentials only
  - Remove: Task Dependencies subsection (already in execute-loop.md lines 251-265)
  - Remove: Parallel Agents subsection (already in execute-loop.md lines 267-275)
  - Remove: E2E-Specific Principles (move to execute-loop.md)
  - Keep: branching, security, tests-with-every-change, CI gate, what-not-how, context mgmt pointer
Verification Gate (5 lines) — compressed to single paragraph
Configuration & References (5 lines) — pointers to config.md and overview.md
```

**Remove entirely from SKILL.md:**
- Mermaid stateDiagram (143 lines) → overview.md
- E2E Project Detection (85 lines) → generate-plans.md
- Test Only Mode (22 lines) → execute-loop.md
- GitHub Planning Stage details (35 lines) → already in github-planning.md
- Plan Frontmatter section (14 lines) → already in config.md
- Assets table (25 lines) → overview.md
- Additional Resources links (12 lines) → already in overview.md
- Changelog (27 lines) → overview.md

### 2. Update overview.md — absorb visual/reference content

**File:** `plugins/pmp/skills/pmp/references/overview.md`

- Add `## Detailed State Diagram` section with the full mermaid stateDiagram from SKILL.md
- Add `## Assets` section with the full template table (Purpose + Used By columns)
- Add `## Changelog` section at the bottom (move from SKILL.md)
- Keep existing simple `graph LR` diagram as quick-glance version

### 3. Update generate-plans.md — absorb E2E Project Detection

**File:** `plugins/pmp/skills/pmp/references/generate-plans.md`

- Add `## E2E Project Detection` section (before step 1, after the Input section) containing:
  - Project Type and E2E Testing Approach table
  - Monorepo Detection table + rules
  - Project Conventions table
  - Execution Model Selection table
  - E2E Test Opt-Out rules
- Update line 108 cross-reference: change `(see E2E Project Detection in SKILL.md)` → `(see E2E Project Detection below)`
- This content was already referenced here — now it lives here

### 4. Update execute-loop.md — absorb execution-time content

**File:** `plugins/pmp/skills/pmp/references/execute-loop.md`

- Add `## Test Only Mode` section (at end, before "When to Stop and Ask"):
  - Move the test runner invocation, results table format, suite selection rules
  - Add note: "For project detection details (type, conventions, monorepo), the plan header carries all detection results from plan generation"
- Add `### E2E Principles` subsection (near top, after the mermaid diagram):
  - Specs not code, structural traceability, two-stage commits, fix loop ceiling, test isolation, no hardcoded conventions
- Add 3 bullets to existing "Parallel Agent Dispatching" section (line 267):
  - Lean prompts — point to CLAUDE.md for project conventions
  - Demand structured returns — reference config.md format
  - Use fast model for reviewers

### 5. Verify github-planning.md — no gaps

**File:** `plugins/pmp/skills/pmp/references/github-planning.md`

Verify these exist (they do — confirmed by reading):
- Source Context requirement: covered in Mandatory Checklist line 759
- Plan annotation steps: covered in Mandatory Checklist lines 800-807
- No content needs to move here; SKILL.md's GitHub Planning Stage was duplication

## Verification

1. Count SKILL.md lines after rewrite — target ≤ 120
2. `grep -r "SKILL.md" plugins/pmp/skills/pmp/references/` — no broken cross-references
3. Read each reference file to confirm it's self-contained for its stage
4. Verify no content was lost — every rule/table exists in exactly one place
