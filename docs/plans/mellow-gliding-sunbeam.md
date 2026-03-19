# Plan: Integrate Single-Source-of-Truth Rules into PMP

## Context

The `docs/single-source-of-truth.md` file defines governance rules for organizing architecture specs: one file per concept, no duplication, structured summary blocks, section-level cross-references, stable anchors, file size limits, etc. These rules should be part of the PMP workflow so that any skill creating or reviewing spec files follows them.

Currently the SSoT doc hardcodes arc42-numbered folder names (e.g., `01-introduction-and-goals/`). The user wants the rules made **folder-structure-agnostic**: use whatever structure the project already has; only apply arc42 numbering when the arc42 skill is explicitly invoked or the project already follows arc42.

## Changes

### 1. Create shared reference: `plugins/pmp/skills/pmp/references/single-source-of-truth.md`

Copy `docs/single-source-of-truth.md` into the PMP shared references hub with these adaptations:

**Folder Structure section (lines 26-53)** — replace the arc42-specific example with generic guidance:
- "Discover the project's existing folder structure first. Adapt to it."
- "If no structure exists, organize into logical section folders that fit the project's domain."
- "Arc42's 12-section structure is one proven option — use it when the project already follows arc42 or when `/pmp:arc42` is invoked."
- Keep the example tree but label it as "Example (arc42-style)" not the default.

**Everything else stays as-is** — the rules about file format, cross-references, naming conventions, duplication elimination, file size limits, self-policing READMEs, etc. are all generic and apply regardless of folder structure.

**Target: ~400 lines** (within 500-line limit)

### 2. Update `plugins/pmp/skills/arc42/SKILL.md`

Add a reference to the SSoT rules in the Workflow section:
- After step 2 (check cache), add: "Read [single-source-of-truth.md](../pmp/references/single-source-of-truth.md) — all reorganized files must follow these formatting and linking rules"
- Add to Key References section

### 3. Update `plugins/pmp/skills/arc42/references/arc42.md`

In Phase 4 (Execution), sections 4b (Create Concern Files) and the Cross-Reference Format Rules:
- Add a reference to SSoT rules for file format (summary block, Depends on, Key sections)
- The arc42 reference already has its own cross-reference rules which are compatible — add a note that SSoT rules apply as the baseline, arc42 adds the numbered-directory conventions on top

### 4. Update `plugins/pmp/skills/spec-review/references/spec-review.md`

In the Orchestration section, after discovery:
- Add SSoT compliance as a check dimension: "Verify spec files follow [single-source-of-truth.md](../../pmp/references/single-source-of-truth.md) formatting rules (summary blocks, cross-references, naming, deduplication)"
- This gives spec-review the ability to flag SSoT violations alongside architecture/security/operations issues

### 5. Update `plugins/pmp/skills/pmp/SKILL.md` (router)

Add SSoT reference to the shared resources listing so any skill can discover it.

### 6. Bump version in `plugins/pmp/.claude-plugin/plugin.json`

`1.8.2` → `1.8.3`

## Files to modify

| File | Action |
|------|--------|
| `plugins/pmp/skills/pmp/references/single-source-of-truth.md` | **Create** — adapted copy from `docs/single-source-of-truth.md` |
| `plugins/pmp/skills/arc42/SKILL.md` | Edit — add SSoT reference to workflow + key references |
| `plugins/pmp/skills/arc42/references/arc42.md` | Edit — reference SSoT file format in Phase 4 |
| `plugins/pmp/skills/spec-review/references/spec-review.md` | Edit — add SSoT compliance check |
| `plugins/pmp/skills/pmp/SKILL.md` | Edit — add SSoT to shared resources |
| `plugins/pmp/.claude-plugin/plugin.json` | Edit — bump to 1.8.3 |

## What NOT to change

- `docs/single-source-of-truth.md` — stays as the standalone reference doc (not part of PMP)
- `plan/` and `execute/` skills — they don't create spec files directly; the SSoT rules apply through arc42 and spec-review which are the spec-touching skills

## Verification

1. Confirm `single-source-of-truth.md` in PMP references is ≤500 lines
2. Confirm arc42 SKILL.md references the new file
3. Confirm spec-review references the new file
4. Confirm no arc42-specific folder structure is forced by default in the shared SSoT copy
5. Read through the modified arc42.md to ensure SSoT file format and arc42 execution rules don't contradict
