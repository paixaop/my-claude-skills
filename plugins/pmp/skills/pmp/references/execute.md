# Execute

Execute implementation plans to completion using agent teams.

**Branch from the detected integration branch before starting. NEVER commit to `main` unless it IS the integration branch.**

Two execution modes: **Subagent-Driven** (fresh agent per task, two-stage review) and **Batch** (execute 3 tasks, report for review). Choose based on user preference.

## Before Starting

1. Create a new branch from the integration branch (read from plan header):
   ```bash
   git checkout <integration-branch> && git pull && git checkout -b feat/<feature-name>
   ```
2. Read the plan file completely
3. Review critically — raise concerns before starting
4. Create TodoWrite with all tasks extracted from the plan

## Mode A: Subagent-Driven Execution

Fresh subagent per task with two-stage review after each. Higher quality, more tokens.

### Per Task

1. **Dispatch implementer subagent** using [implementer-prompt.md](implementer-prompt.md) template
   - Provide FULL task text (don't make subagent read plan file)
   - Include scene-setting context: where task fits, dependencies, architecture
2. **If subagent asks questions:** Answer clearly and completely before they proceed
3. **Subagent implements, tests, commits, self-reviews**
4. **Dispatch spec reviewer** using [spec-reviewer-prompt.md](spec-reviewer-prompt.md) template
   - Reviewer reads actual code — does NOT trust implementer's report
5. **If spec issues found:** Implementer fixes, spec reviewer re-reviews. Repeat until approved
6. **Dispatch code quality reviewer** using [code-quality-reviewer-prompt.md](code-quality-reviewer-prompt.md) template
   - Only AFTER spec compliance passes
7. **If quality issues found:** Implementer fixes, quality reviewer re-reviews. Repeat until approved
8. **Mark task complete** in TodoWrite

### Red Flags

NEVER:
- Skip either review stage (spec compliance AND code quality required)
- Start quality review before spec compliance passes
- Dispatch multiple implementers in parallel (conflicts)
- Make subagent read plan file (provide full text)
- Accept "close enough" on spec compliance
- Skip re-review after fixes
- Move to next task with open review issues

### After All Tasks

- Dispatch final code reviewer for the entire implementation
- Run detected CI command — must pass clean
- Announce with completion message from [config.md](../config.md) Stage Announcements

## Mode B: Batch Execution

Execute tasks in batches (see [config.md](../config.md) Thresholds for batch size), report for review between batches. Lighter weight, human-in-loop.

### Per Batch

1. **Execute next 3 tasks** sequentially
2. For each task:
   - Follow each step exactly as written in the plan
   - Run verification commands as specified
   - Check phase exit criteria before proceeding
   - Commit after each task
3. **After batch:** Report what was implemented with verification output
4. **Wait for feedback** — apply changes if needed
5. **Next batch** or complete

### After All Batches

- Run detected CI command — must pass clean
- Update plan file: mark **fully implemented** or **partially implemented** (noting remaining steps)
- Announce with completion message from [config.md](../config.md) Stage Announcements

## Verification After Every Task

After each task, regardless of execution mode, run the project's verification commands (from plan header):

1. **Build** — project compiles/builds successfully
2. **Lint** — linter passes clean
3. **Test** — full test suite passes
4. **Complexity** — within project limits

After the final task: run the detected CI command for a full clean pass.

Do NOT proceed to the next task if any check fails. Fix first.

## When to Stop and Ask

**STOP immediately when:**
- Missing dependency or unclear requirement
- Test fails repeatedly after fix attempts
- Plan has critical gaps
- You don't understand an instruction
- Verification fails and you can't determine why

Ask for clarification rather than guessing.

## Completion

After all tasks complete and CI passes:

1. Run the detected CI command one final time — show the output as evidence
2. **Ask the user:** "All tasks complete and CI passes. Want me to create a PR?"
3. **If yes:** Create the PR using `gh pr create` with a summary of what was implemented
4. **Stamp the plan file:** Prepend `Fully implemented <PR#> <timestamp>` at the top of the plan file
5. **Archive the plan:** Move plan file(s) from plans to completed plans directory (see [config.md](../config.md) File Paths; create directory if it doesn't exist)
6. Announce with completion message from [config.md](../config.md) Stage Announcements
