> **This file continues from [plan-lod.md](plan-lod.md)** — see that file for Phases 1-3 (Clarify Agent Boundaries, LOD Architecture + Shared Boilerplate, Unify Phase/Wave Terminology).

## Phase 4: Slim Redundant Entry Points

### 4a. Slim full.md

**[full.md](full.md)** -- Replace the current 52-line file with a ~20-line version:

```markdown
---
description: "Complete 9-phase offensive security analysis. No prompts — runs everything."
model: opus
---

# Security Analyst — Full Analysis

Run the complete analysis with no interactive prompts.

**Settings:**
- Scope: Full — all phases, all attack surfaces
- Output: Files to `docs/security/runs/{YYYY-MM-DD-HHMMSS}/`
- Agent skipping: Only if recon determines irrelevance (e.g., no frontend)

**Execution:** Follow the orchestration flow in `security-analyst.md` with these settings.

**Output files:** See `security-analyst.md` Step 9 for the full file list.

**Expected duration:** 23+ agents across 6 execution groups. Significant processing time.
```

### 4b. Slim recon.md

**[recon.md](recon.md)** -- Replace with ~20-line version:

```markdown
---
description: "Quick security reconnaissance — maps attack surface without vulnerability analysis."
model: opus
---

# Security Analyst — Reconnaissance Only

Run Phase 0 only. Produces a structured security map without vulnerability analysis.

**Execution:** Follow `security-analyst.md` Steps 1-2 (create run dir, spawn recon agent). Stop after recon completes.

**Output:** `docs/security/runs/{YYYY-MM-DD-HHMMSS}/recon-report.md`

**Next steps:** `/security-analyst:full` for complete analysis, `/security-analyst:focused [component]` to target a specific area.
```

---

## Phase 5: Add Missing Pieces

### 5a. Create threat-model agent prompt

**Create [prompts/threat-model-agent.md**](prompts/threat-model-agent.md) (~80-100 lines):

- Accept `{RECON_REPORT_PATH}` and `{OUTPUT_PATH}` as inputs
- Task 1: STRIDE analysis for each trust boundary from the recon report
- Task 2: Attack trees for each crown jewel (root = "Compromise X", branches = attack paths)
- Task 3: Risk matrix (likelihood x impact for each threat)
- Output: Write `{OUTPUT_PATH}` (threat-model.md)
- Use the same offensive framing as other agents
- Follows LOD output conventions for any findings discovered during threat modeling

Update **[threat-model.md](threat-model.md)** execution step 3 to spawn this agent via Task tool.

### 5b. Add plugin conflict guidance

**[plugins/README.md](plugins/README.md)** -- Add a new section after "Creating a New Plugin":

```markdown
## Plugin Overlap and Conflicts

When multiple plugins inject sections for the same agent (e.g., both Next.js and React inject `## attack-surface-frontend`), the orchestrator concatenates all matching sections. Agents should:

- Treat multiple plugin sections as **complementary** checks — run all of them
- If checks conflict, prefer the **more specific** plugin (e.g., Next.js over React for Next.js projects)
- Report the plugin source in findings so the orchestrator can trace which plugin triggered the check
```

---

## Phase 6: Update SKILL.md

**[SKILL.md](SKILL.md)** -- Three additions:

### 6a. Update directory listing (lines 149-195) to include:

- `templates/agent-common.md` (shared LOD output + incidental findings instructions)
- `prompts/threat-model-agent.md`

### 6b. Add LOD architecture documentation after "Key Principles":

```markdown
### Level of Detail (LOD) Architecture

Agent reports use a three-tier Level of Detail system to minimize context consumption across independent agent context windows:

| Level | Content | ~Tokens | Where it lives |
|---|---|---|---|
| LOD-0 | `ID \| Severity \| One sentence` | 30/finding | Agent return message, orchestrator context, `{PRIOR_FINDINGS_SUMMARY}` |
| LOD-1 | ID + title + severity + CVSS + CWE + file:line + paragraph | 100/finding | `{FINDINGS_DIR}/index.md`, terminal agent prompts |
| LOD-2 | Full finding with PoC, remediation, regression test | 800/finding | `{FINDINGS_DIR}/{FINDING-ID}.md`, read on demand |

**How it flows:**
1. Analysis agents write atomic LOD-2 files to `{FINDINGS_DIR}/` and return LOD-0 summaries
2. The orchestrator builds `{FINDINGS_DIR}/index.md` with LOD-0 + LOD-1 views
3. Next-group agents receive LOD-0 in their prompt; selectively `Read` LOD-2 files for relevant findings
4. Terminal agents (exploit dev, critic, report writer) receive LOD-1 index; `Read` all LOD-2 files

This saves 80-96% of prompt tokens for downstream agents while preserving full detail on disk.
```

### 6c. Add context budget note (replaces the simpler version from old Phase 5d):

```markdown
### Context Budget Considerations

The LOD architecture mitigates most context pressure, but be aware:
- The LLM agent prompt (`attack-surface-llm.md`) is ~340 lines before plugin injection — the largest single prompt
- Terminal agents (exploit dev, critic, report writer) still read all LOD-2 files via tool calls; for runs with 40+ findings, consider batching reads
- Plugin sections add to every injected agent's context — keep plugins concise
```

---

## Disk Structure After Changes

```
{RUN_DIR}/
├── findings/                         # NEW: atomic LOD-2 finding files
│   ├── index.md                      # NEW: LOD-0 table + LOD-1 briefs (built by orchestrator)
│   ├── P1-HTTP-001.md               # Written by surface-http agent
│   ├── P1-HTTP-002.md
│   ├── P2-INJ-001.md               # Written by history-injections agent
│   └── ...
├── recon-report.md                   # Unchanged
├── group-1-attack-surface.md        # RENAMED from wave-1, now contains LOD-0 table + refs
├── group-2-business-logic.md        # RENAMED from wave-2
├── group-3-data-flow.md             # RENAMED from wave-3
├── group-4-exploits.md              # RENAMED from wave-4
├── group-4.5-critic-review.md       # RENAMED from wave-4.5
├── security-analyst-final-report.md          # RENAMED from wave-5
├── fix-plan.md                       # Unchanged
└── threat-model.md                   # If threat-model mode used
```

---

## Files Changed Summary


| File                                | Change Type                                                                  | Phase      |
| ----------------------------------- | ---------------------------------------------------------------------------- | ---------- |
| `prompts/attack-surface-http.md`    | Edit (boundary directives + LOD output)                                      | 1a, 1b, 2e |
| `prompts/attack-surface-authz.md`   | Edit (remove Task 6 + LOD output)                                            | 1a, 2e     |
| `prompts/logic-authz-escalation.md` | Edit (prior-findings note + LOD output)                                      | 1a, 2e     |
| `prompts/logic-pipeline-exploit.md` | Edit (conditional defer + LOD output)                                        | 1c, 2e     |
| `prompts/git-history-injections.md` | Edit (defer AI injection + LOD output)                                       | 1c, 2e     |
| `prompts/exploit-developer.md`      | Edit (self-check clarify + LOD input)                                        | 1d, 2f     |
| `prompts/finding-critic.md`         | Edit (LOD-aware input)                                                       | 2f         |
| `prompts/report-writer.md`          | Edit (LOD-aware input)                                                       | 2f         |
| `prompts/fix-planner.md`            | Edit (LOD output)                                                            | 2e         |
| Remaining 11 prompt files           | Edit (LOD output + incidental placeholder)                                   | 2e         |
| `templates/finding.md`              | Edit (add LOD-0, LOD-1, LOD-2 definitions)                                   | 2a         |
| `templates/agent-common.md`         | **Create** (shared LOD output + incidentals)                                 | 2b         |
| `templates/wave-report.md`          | Edit (LOD-0 table + refs)                                                    | 2d         |
| `security-analyst.md`               | Edit (FINDINGS_DIR, LOD spawning, consolidation, mapping table, wave->group) | 2c, 3a, 5b |
| `SKILL.md`                          | Edit (LOD docs, group rename, context budget, directory listing)             | 3b, 6      |
| `full.md`                           | Rewrite (slim + group rename)                                                | 3c, 4a     |
| `recon.md`                          | Rewrite (slim)                                                               | 4b         |
| `prompts/threat-model-agent.md`     | **Create**                                                                   | 5a         |
| `threat-model.md`                   | Edit (use new agent prompt)                                                  | 5a         |
| `plugins/README.md`                 | Edit (add conflict guidance)                                                 | 5b         |


**Total: 30 file edits, 2 new files created**
