# Fix PMP Plugin Inconsistencies

## Context

A full review of the PMP plugin found an internal inconsistency where the plan-review checklist doesn't acknowledge the E2E opt-out mechanism, and a stale skill description for spec-implementability. The line-count violations are being left as-is per user decision.

---

## Issue 1: Plan-review E2E check ignores opt-out logic

**File:** `plugins/pmp/skills/plan-review/references/review.md`

**Problem:** Lines 127-130 have an E2E testing checklist item that asks the reviewer to flag missing E2E tests. But `generate-plans.md` (lines 75-101) defines a legitimate E2E Test Opt-Out mechanism for certain project types (utility libraries, type-only packages, docs-only changes, infra changes). A reviewer following the checklist would incorrectly flag plans that went through the opt-out.

**Fix:** Insert one bullet after line 130 (after the existing three sub-bullets):
```
  - If the plan explicitly documents an E2E opt-out decision (per generate-plans.md § E2E Test Opt-Out), verify the opt-out reason is valid for the project type — do not flag as missing
```

---

## Issue 2: Stale "11-criteria" in spec-implementability description

**File:** `plugins/pmp/skills/spec-implementability/SKILL.md`

**Problem:** The system reminder shows "11-criteria production-readiness gate" but the actual skill has 13 criteria (12 and 13 were added). The body text (line 10) correctly says "13-criteria" but the frontmatter `description` (line 3) is trigger-phrase only and doesn't include the count. A previous version's description included "11-criteria" and that got cached.

**Fix:** Update the frontmatter `description` on line 3 to include "13-criteria":
```
description: Implementability assessment — 13-criteria production-readiness gate. Use when the user says 'is this implementable', 'ready to code', 'spec completeness', 'can we implement this', or wants to verify a spec is complete enough for a coding agent to implement without clarification
```

---

## Version Bump

Bump `plugins/pmp/.claude-plugin/plugin.json` version from `1.8.5` → `1.8.6`.

---

## Files to Modify

| File | Action |
|------|--------|
| `plugins/pmp/skills/plan-review/references/review.md` | Add E2E opt-out acknowledgment bullet after line 130 |
| `plugins/pmp/skills/spec-implementability/SKILL.md` | Update frontmatter description to include "13-criteria" |
| `plugins/pmp/.claude-plugin/plugin.json` | Version bump 1.8.5 → 1.8.6 |

---

## Verification

1. Read updated `review.md` lines 127-132 — confirm the new bullet flows naturally with existing checklist
2. Read updated `spec-implementability/SKILL.md` frontmatter — confirm description includes "13-criteria"
3. Confirm `plugin.json` version is `1.8.6`
