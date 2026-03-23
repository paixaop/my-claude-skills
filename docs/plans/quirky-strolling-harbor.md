# Fix: Redundant File Reading in PMP Review Skills

## Context

The spec-review (architecture review) and plan review skills burn excessive tokens because of redundant file reading. The main controller reads ALL files, then spawns agents that each re-read ALL the same files. For a 20-file spec review with 3 agents, that's 4x the token cost.

Root cause: Both SKILL.md files say "Use agent teams (Task tool)" but agents don't share context with the main controller — each starts fresh and re-reads everything. Passing file contents inline to agents doesn't help either, since it duplicates the same tokens N times (once per agent prompt).

## Approach: No Agents for Analysis

**Agents can only be used for parallel file reading, not for analysis phases.** All analysis runs sequentially in the main controller, which already has full file context from the reading phase. This eliminates all redundant file reads.

- **Parallel agents**: allowed ONLY during initial file reading (Phase 0 / Step 2) to speed up I/O when there are many files
- **Analysis phases**: all run sequentially in the main controller context — no agent dispatch
- **Files read once**: the main controller reads every file exactly once, then all phases operate on that context

---

## Changes

### 1. Remove agent dispatch from spec-review analysis phases

**File:** [spec-review.md](../../plugins/pmp/skills/spec-review/references/spec-review.md)

Add a section after Phase 0 (Discovery), before Phase 1:

```markdown
## Context Management

**All analysis phases (1–14) run in the main controller context — do NOT spawn agents for analysis.**

Agents share no context with the main controller. Spawning agents for analysis phases forces each agent to re-read all files, multiplying token cost by the number of agents. Instead:

- **Phase 0 (Discovery):** Use parallel agents ONLY for reading files when the corpus is large (10+ files). Each agent reads a partition of files and returns the content. The main controller collects all file contents.
- **Phases 1–14 (Analysis):** Run sequentially in the main controller. The controller already has all file contents from Phase 0 — no re-reading needed.
- **Phase 14 (Remediation):** Naturally cross-references findings from all phases since everything is in one context.

### Large Corpus Fallback: Batched Phases

When the corpus is large enough to risk context overflow (estimated >30 files or >50K words of source), split analysis into batches:

1. **Batch 1:** Phase 0 (Discovery) + Phase 1 (System Reconstruction) + Phases 2-5 (Architecture, Consistency, Invariants, State Machines)
2. **Batch 2:** Phases 6-7 (Threat Modeling, Attack Simulation) + Phase 13 (AI Red Team, if applicable)
3. **Batch 3:** Phases 8-12 (Performance, Resources, Failure Modes, Scalability, Operability) + Phase 14 (Remediation)

Between batches, produce a **checkpoint summary**:
- Key findings from completed phases (structured, ~500 words)
- The system model from Phase 1 (components, boundaries, state — ~300 words)
- List of files most relevant to the next batch

Each new batch starts by re-reading the source files + the checkpoint. This means files are re-read 2-3x total instead of N-agents × all-files. The checkpoint ensures cross-phase coherence.

**Batch boundaries are phase-aware:** never split a logically connected pair (e.g., threat modeling + attack simulation stay together).
```

### 2. Remove agent dispatch from plan review analysis

**File:** [review.md](../../plugins/pmp/skills/review/references/review.md)

**Step 2 (line 155)** — change:

From:
> Read ALL referenced code files — use agent teams for parallel reading if many files

To:
> **Read ALL referenced code files.** Use parallel agents only for the file reading itself when there are many files (10+) — each agent reads a partition of files and returns content. All subsequent analysis (checklist scoring, security gate, rewriting) runs in the main controller context using the already-loaded file contents. Do NOT spawn agents for analysis — they would re-read all files, wasting tokens.

**Step 4 (Security gate, line 160)** — add note:

> The security analysis runs in the main controller context, which already has all code files loaded from Step 2. Do not re-read files for the security gate — work from the file contents already in context.

### 3. Update SKILL.md files to clarify agent usage

**File:** [spec-review SKILL.md](../../plugins/pmp/skills/spec-review/SKILL.md) (line 12)
**File:** [review SKILL.md](../../plugins/pmp/skills/review/SKILL.md) (line 12)

Change:
> Use agent teams (Task tool) and track progress with TodoWrite throughout.

To:
> Use agent teams (Task tool) ONLY for parallel file reading when the corpus is large. All analysis runs in the main controller context — do not spawn agents for analysis phases (they would re-read all files). Track progress with TodoWrite throughout.

### 4. Add agent constraint to config.md

**File:** [config.md](../../plugins/pmp/skills/pmp/config.md)

Add to Context Management section (after line 150):

```markdown
### Agent Usage Rules

Agents share no context with the main controller — each starts with an empty context window. This means:

- **DO use agents** for parallel file reading (I/O-bound work where each agent reads a different set of files)
- **DO NOT use agents** for analysis phases that need access to already-read file contents (they would re-read everything, multiplying token cost)
- **Exception:** An agent is justified only when it needs to read files that are NOT already in the main context (e.g., a targeted deep-dive into files not part of the initial corpus)
```

---

## Token Impact

| Scenario | Before | After | Reduction |
|----------|--------|-------|-----------|
| spec-review, 20 files, 3 agents | ~80 reads (4×20) | 20 reads (1×20) | 75% |
| plan review, 15 code files | ~30 reads (2×15) | 15 reads (1×15) | 50% |

## Verification

1. Run `/pmp:spec-review` on a project with 10+ spec files — verify no agents are spawned during Phases 1-14, only during Phase 0 file reading
2. Run `/pmp:review` on a plan referencing 10+ code files — verify security gate doesn't re-read files
3. Check that analysis quality is maintained — findings should be comparable
4. Monitor token usage — should see significant reduction

## Files to Modify

1. [spec-review.md](../../plugins/pmp/skills/spec-review/references/spec-review.md) — Add Context Management section after Phase 0
2. [review.md](../../plugins/pmp/skills/review/references/review.md) — Update Step 2 and Step 4
3. [spec-review SKILL.md](../../plugins/pmp/skills/spec-review/SKILL.md) — Clarify agent usage
4. [review SKILL.md](../../plugins/pmp/skills/review/SKILL.md) — Clarify agent usage
5. [config.md](../../plugins/pmp/skills/pmp/config.md) — Add Agent Usage Rules
