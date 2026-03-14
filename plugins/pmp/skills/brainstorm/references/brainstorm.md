# Brainstorm

Turn ideas into validated designs through collaborative dialogue. This stage **loops** until the user is satisfied.

**HARD-GATE:** Do NOT write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it. This applies to EVERY project regardless of perceived simplicity.

## Process

You MUST create a TodoWrite task for each step and complete them in order:

1. **Explore project context** — check files, docs, recent commits, CLAUDE.md
2. **Brainstorm loop** — iterate with the user:
   a. **Ask clarifying questions** — one at a time, understand purpose/constraints/success criteria
   b. **Propose 2-3 approaches** — with trade-offs and your recommendation
   c. **Present design sections** — scaled to complexity, get user approval after each section
   d. **Refine** — incorporate feedback, revisit earlier decisions if needed
   e. **Check exit** — if the user says to move ahead, exit the loop. Otherwise, after presenting a complete design, use AskQuestion: "The design covers [summary]. Ready to move to plan generation, or want to refine further?"
3. **Write design doc** — save to plans directory using design doc filename pattern from [config.md](../../pmp/config.md) File Paths
4. **Transition to Generate Plan** — read [generate-plans.md](../../plan/references/generate-plans.md) and follow it

## Exploring Context

Before asking questions:
- Read CLAUDE.md for architecture and conventions
- Check `docs/plans/` for existing plans and designs
- Scan recent git log for relevant changes
- Identify related code files

Use agent teams for parallel research when multiple areas need exploration:
- **codebase-locator**: Find related files
- **codebase-analyzer**: Understand current implementation
- **docs-locator**: Find existing docs about this feature

## Asking Questions

- **One question per message** — don't overwhelm
- **Prefer multiple choice** when possible (use AskQuestion tool)
- Focus on: purpose, constraints, success criteria, edge cases
- If a topic needs more exploration, break into multiple questions

## Proposing Approaches

- Always propose **2-3 approaches** with trade-offs
- Lead with your recommendation and explain why
- Address security implications for each approach
- Consider impact on existing architecture (see CLAUDE.md file layout)

## Presenting the Design

- Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing, security
- Apply YAGNI ruthlessly — remove unnecessary features

## Writing the Design Doc

Save using design doc filename pattern from [config.md](../../pmp/config.md) File Paths, using [assets/design-doc.md](../assets/design-doc.md).

## Transition

After the design doc is saved:
1. Use AskQuestion: "Design saved. Ready to generate the implementation plan?"
2. Once confirmed, read [generate-plans.md](../../plan/references/generate-plans.md) and follow it — the design doc becomes input
3. Do NOT invoke any other skill or start implementation
