> **This file continues from [`security-analyst.md`](security-analyst.md)** — see that file for frontmatter, critical rules, orchestration model, initial setup, constants, and Steps 1-5.

### Step 6: TRACING STAGE — Phase 4 (data flow tracing, needs surface and logic)

Spawn flow tracers based on `{RECON_DIR}/step-09-data-flows.md`.

Create tasks for each critical data flow (up to 4):

```
TaskCreate: subject="Tracing: Data flow — {flow_name}", activeForm="Tracing {flow_name}",
  description="[data-flow-tracer prompt with TRACE_TARGET={flow_name}]"
```

For each critical data flow:
- `flow-tracer-N` — `{PROMPTS_DIR}/data-flow-tracer.md`
- Set `{TRACE_TARGET}` to the specific flow name
- Include surface and logic stage findings as prior findings

**Spawn flow-tracer agents:** Issue all Task tool calls in one message with self-contained prompts.

**Wait:** Each Task call returns when done.
Verify expected output: confirm finding files were written to `{FINDINGS_DIR}/`.
Collect LOD-0 summaries from Task return values, consolidate. **Claude Code / Codex:** Mark tasks completed via TaskUpdate.

Write tracing stage findings to `{REPORTS_DIR}/tracing.md` using `{GROUP_REPORT_TEMPLATE}`:
- `{GROUP_NAME}` → "Tracing: Data Flow Tracing"
- `{PRIOR_GROUP_FILES}` → "recon/index.md, reports/surface.md, reports/logic.md"

### Step 7: EXPLOITS STAGE — Phase 7 (exploit development, needs all findings)

```
TaskCreate: subject="Exploits: Exploit development", activeForm="Developing exploits",
  description="[exploit-developer prompt with ALL_FINDINGS replaced]"
```

Spawn a single agent:
- `exploit-dev` — `{PROMPTS_DIR}/exploit-developer.md`
- `{ALL_FINDINGS}` → consolidated findings from the surface through tracing stages (Medium+ only)

Agent executes, writes findings, returns catalog in its response.

**Wait:** Task call returns when done.
Verify expected output: confirm exploit catalog was written to `{FINDINGS_DIR}/`.
Collect exploit catalog from Task return value. **Claude Code / Codex:** Mark task completed via TaskUpdate.

Write exploits stage findings to `{REPORTS_DIR}/exploits.md` using `{GROUP_REPORT_TEMPLATE}`:
- `{GROUP_NAME}` → "Exploits: Exploit Development"
- `{GROUP_FINDINGS}` → complete exploit catalog
- `{PRIOR_GROUP_FILES}` → all prior stage files

### Step 7.5: VALIDATION STAGE — Finding Validation (Critic)

The critic agent adversarially reviews ALL findings from exploit development.

```
TaskCreate: subject="Validation: Finding validation", activeForm="Validating findings",
  description="[finding-critic prompt with ALL_FINDINGS replaced]"
```

1. Read `{PROMPTS_DIR}/finding-critic.md`
2. Spawn agent via Task tool:
   - name: "finding-critic"
   - subagent_type: "generalPurpose"
   - prompt: Content of finding-critic.md with placeholders + "Return validated findings in your response."
     - `{RECON_INDEX_PATH}` → absolute path to recon index
     - `{RECON_DIR}` → absolute path to recon directory
     - `{ALL_FINDINGS}` → complete exploit catalog from the exploits stage
     - `{FINDING_TEMPLATE_PATH}` → absolute path to finding template
3. **Wait:** Task call returns when done
4. Process critic results:
   - Remove findings marked REMOVED
   - Update severity for DOWNGRADED/UPGRADED findings
   - Replace fixes where critic provided corrections
   - Log stats: X confirmed, Y downgraded, Z removed, W upgraded
5. **Claude Code / Codex:** Mark task completed via TaskUpdate

Write validation stage findings to `{REPORTS_DIR}/validation.md` using `{GROUP_REPORT_TEMPLATE}`:
- `{GROUP_NAME}` → "Validation: Finding Validation (Critic Review)"
- `{GROUP_FINDINGS}` → validated catalog with annotations (CONFIRMED/DOWNGRADED/REMOVED/UPGRADED)
- Include critic stats (X confirmed, Y downgraded, Z removed, W upgraded) in the Notes section

**If scope is "Focused":** Skip the validation stage — present findings directly after exploit development. **STOP — do not continue to report writing. Wait for user follow-up.**

### Step 8: REPORTING STAGE — Phase 8 (final report, needs everything)

```
TaskCreate: subject="Reporting: Final report", activeForm="Writing final report",
  description="[report-writer prompt with ALL_FINDINGS and CRITIC_STATS replaced]"
```

Spawn a single agent:
- `report-writer` — `{PROMPTS_DIR}/report-writer.md`
- `{ALL_FINDINGS}` → validated exploit catalog from the validation stage (critic-reviewed)
- `{CRITIC_STATS}` → summary of critic results (confirmed/downgraded/removed/upgraded counts)
- `{FINAL_REPORT_TEMPLATE_PATH}` → absolute path to final report template
- `{OUTPUT_PATH}` → absolute path to final report

**Wait:** Task call returns when done.
Verify expected output: confirm `{FINAL_REPORT}` was written.
**Claude Code / Codex:** Mark task completed via TaskUpdate.

### Step 8.5: Phase 9 — Fix Plan Generation

```
TaskCreate: subject="Remediation: Fix plan generation", activeForm="Generating fix plan",
  description="[fix-planner prompt with REPORT_PATH replaced]"
```

Spawn the fix-planner agent to create an implementation plan from the final report.

1. Read `{PROMPTS_DIR}/fix-planner.md`
2. Read `{FIX_PLAN_TEMPLATE}`
3. Spawn agent via Task tool:
   - name: "fix-planner"
   - subagent_type: "generalPurpose"
   - prompt: Content of fix-planner.md with placeholders + "Write output to {OUTPUT_PATH}. Return confirmation in your response."
     - `{REPORT_PATH}` → absolute path to `{FINAL_REPORT}`
     - `{FIX_PLAN_TEMPLATE_PATH}` → absolute path to fix-plan template
     - `{FINDING_TEMPLATE_PATH}` → absolute path to finding template
     - `{OUTPUT_PATH}` → `{REPORTS_DIR}/fix-plan.md`
     - `{RUN_DIR}` → the run directory path
4. **Wait:** Task call returns when done
5. Verify expected output: confirm `{REPORTS_DIR}/fix-plan.md` was written
6. **Claude Code / Codex:** Mark task completed via TaskUpdate

### Step 9: Cleanup & Termination

1. Present summary to user:
   - **Run directory:** `{RUN_DIR}`
   - **Files generated:**
     - `{RUN_DIR}/recon/index.md` (LOD-0 + LOD-1 recon index)
     - `{RUN_DIR}/recon/step-*.md` (14 atomic LOD-2 recon section files)
     - `{RUN_DIR}/findings/` (atomic LOD-2 finding files)
     - `{RUN_DIR}/findings/index.md` (LOD-0 + LOD-1 index)
     - `{REPORTS_DIR}/surface.md`
     - `{REPORTS_DIR}/sbom.md` (Software Bill of Materials)
     - `{REPORTS_DIR}/logic.md`
     - `{REPORTS_DIR}/tracing.md`
     - `{REPORTS_DIR}/exploits.md`
     - `{REPORTS_DIR}/validation.md`
     - `{REPORTS_DIR}/final.md`
     - `{REPORTS_DIR}/fix-plan.md`
   - **Plugins loaded:** list of matched plugins (or "None")
   - Total findings by severity
   - Top 3 most critical findings
   - Critic validation results (findings confirmed vs rejected)
   - Fix plan summary: X tasks, Y quick wins, Z estimated total effort

**STOP. The analysis is complete. Do not take any further actions, spawn any more agents, or continue reasoning. Your final message to the user is the summary above. Wait for the user to ask a follow-up question before doing anything else.**

## Consolidation Between Stages

After each stage:
1. Confirm all Task calls for the stage have returned. **Claude Code / Codex:** Verify all stage tasks show status: completed via TaskList.
2. Verify expected output: Glob `{FINDINGS_DIR}/` to confirm atomic finding files were written
3. Collect LOD-0 summary tables from all agents in the stage
4. Deduplicate: if two agents wrote findings for the same vulnerability, keep the more detailed LOD-2 file and delete the other. Update LOD-0 tables accordingly
4. Build/update `{FINDINGS_INDEX}`:
   - **LOD-0 section**: Concatenated LOD-0 tables from all stages so far
   - **LOD-1 section**: For each finding, read its LOD-2 file and extract the LOD-1 brief (first 5 lines: ID, title, severity, CVSS, CWE, ATT&CK, file:line, one-paragraph)
5. Enrich: add cross-references between related findings in the index
6. Prepare `{PRIOR_FINDINGS_SUMMARY}` for next stage: the LOD-0 table only, plus `{FINDINGS_DIR}` path for selective reading
7. Collect incidental findings (IX-prefixed) from agent return messages
8. Write the stage report using `{GROUP_REPORT_TEMPLATE}`

### Known Overlap Areas

When deduplicating between stages, pay special attention to:

- **IDOR/AuthZ**: The surface stage's HTTP and AuthZ agents flag suspect endpoints (shallow). The logic stage's authz-escalation agent deepens the analysis (LOD-2). Merge by keeping the deeper logic stage findings and removing surface stage flags that were fully investigated.
- **Rate limiting/DoS**: The surface stage's HTTP agent flags missing rate limits. The logic stage's DoS agent assesses amplification and cost. Merge by promoting the DoS agent's finding and removing the HTTP agent's bare flag.
- **AI/LLM**: The surface stage's LLM agent covers OWASP categories. The logic stage's pipeline agent covers downstream pipeline impact. These are complementary — keep both, cross-reference in the index.

## Parallelism Rules

**Maximize wall-clock efficiency by batching all independent work:**

1. **Batch agent spawns**: When spawning N independent agents for a stage, issue ALL N Task tool calls in a single message. Never spawn agents one-at-a-time when they are independent.
2. **Batch file reads**: When preparing prompts for a stage, read all prompt files, templates, and plugin files in a single batch of Read tool calls before constructing any prompts.
3. **Overlap independent phases**: SBOM assembly (Step 4.5) and the logic stage (Step 5) are independent — start both after the surface stage completes, don't wait for SBOM before starting the logic stage.
4. **Pipeline prep during execution**: While waiting for a stage to complete, pre-read prompt templates needed for the next stage so they're ready to inject immediately.
5. **Respect stage dependencies**: Stages MUST run in dependency order (recon→surface→logic→tracing→exploits→validation→reporting→remediation). Within a stage, all agents run in parallel.

## Agent Spawning Pattern

For each stage of agents, the spawning pattern is:

```
PREPARATION (batch all reads in one message):
1. Read ALL prompt templates for this stage: Read {PROMPTS_DIR}/{prompt-file}.md (batch)
2. Read the finding template: Read {TEMPLATES_DIR}/finding.md
3. Read the shared output template: Read {TEMPLATES_DIR}/agent-common.md
4. Collect plugin sections (see Step 3.5)

TASK CREATION (Claude Code / Codex only — skip on Cursor):
5. For each agent, create a task via TaskCreate:
   - subject: "{StageName}: {agent description}"
   - activeForm: "{present participle of agent action}"
   - description: Constructed prompt with ALL placeholders replaced

PROMPT CONSTRUCTION (for each agent):
6. Construct the agent prompt by replacing ALL placeholders:
   - {RECON_INDEX_PATH} → absolute path to recon index
   - {RECON_DIR} → absolute path to recon directory
   - {FINDING_TEMPLATE_PATH}, {PRIOR_FINDINGS_SUMMARY}, {PLUGIN_CHECKS} (existing)
   - {FINDINGS_DIR} → absolute path to findings directory
   - {INCIDENTAL_FINDINGS_SECTION} → content from agent-common.md
   - {AGENT_PREFIX} → agent's finding ID prefix (e.g., HTTP, AUTHZ, INJ)
   - {AGENT_NAME} → agent's display name
   - Append the agent-common.md output instructions to the end of the prompt

SPAWNING:
7. Spawn each agent via Task tool with full self-contained prompt. Agent writes to disk, returns LOD-0 in response. Batch all into ONE message for maximum parallelism.

MONITORING:
8. Each Task call blocks until agent completes. Collect LOD-0 from return value.
9. Verify expected output was written to disk
10. **Claude Code / Codex:** Mark each task as completed via TaskUpdate
```

## Focused Analysis Mode

If the user selects "focused on [component]":
1. Run the recon stage — always needed
2. Read recon index and relevant step files to identify which phases/agents are relevant:
   - "authentication" → surface-http (auth endpoints), surface-authz, history-auth-bypass, logic-races, logic-authz, flow-tracer (auth flow)
   - "data processing pipeline" → surface-http (webhooks), surface-integrations, logic-pipeline, logic-dos, flow-tracer (pipeline flow)
   - "frontend" → surface-frontend, config-infra (CSP/CORS), history-data-exposure
   - "database rules" → surface-authz, logic-authz
   - "AI integration" → surface-llm, surface-integrations, logic-pipeline, history-injections
   - "CI/CD" → cicd-pipeline, config-infra
   - "containers" → container-security, config-infra
   - "API" → surface-http, api-schema, logic-authz
   - "real-time" → websocket-security, logic-dos, logic-races
   - "file uploads" → file-upload-security, surface-http, config-infra
   - "privacy" → (use `/security-analyst:privacy` instead)
3. Run only relevant agents, still in stage order (respect dependencies)
4. Skip exploit development and report writing for focused mode — present findings directly

## Variant Hunt Mode

If the user selects "variant hunt for [vuln]":
1. Run the recon stage
2. Run Phase 2 agents (git history) focused on the specific vulnerability type
3. Run Phase 3 agents where relevant (especially race conditions and pipeline)
4. Skip Phases 1, 5, 6, 8
5. Present variant findings directly

## Important Notes

1. **All agents are general-purpose** — they need Bash (for git log, git diff, npm audit), Read, Grep, Glob for code analysis. Explore agents are read-only and cannot run Bash.
2. **Recon data is on disk** — the recon index and step files are written to `{RECON_DIR}/` so agents can selectively Read relevant sections without bloating their context.
3. **Findings collection:** LOD-0 comes from the Task tool return value. Full LOD-2 findings are on disk. The orchestrator consolidates between stages.
4. **Dynamic agent count** — skip irrelevant agents based on recon findings. Don't spawn a frontend agent for a backend-only project or an LLM agent for a project without AI integration.
5. **Stage discipline** — never start a stage before the previous stage completes. Later stages depend on earlier findings.
6. **Task tool blocks** — each Task call blocks until the agent completes. No polling or messaging needed.
7. **Incidental findings** — All agents report security issues outside their focus area with IX- prefix. The orchestrator collects these between stages and includes them in the catalog. Incidentals don't block stages.
