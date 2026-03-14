---
name: pmp
version: "1.7.1"
description: "Full planning lifecycle router — dispatches to focused sub-skills for each stage. Use this when the user's intent is ambiguous or spans multiple stages, e.g. 'plan this feature', 'help me build X', 'I have a feature idea', or general planning requests. For specific stages, prefer the focused sub-skills: pmp:brainstorm (design exploration), pmp:plan (generate plans), pmp:review (plan review), pmp:execute (code-test-fix), pmp:spec-review (architecture analysis), pmp:github (issues/projects), pmp:decompose (phase breakdown), pmp:changelog (release notes). This root skill routes to the right stage based on the user's input."
---

# PMP — Plan

Full planning lifecycle: brainstorm, write, review, execute. Routes to focused sub-skills for each stage.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

**CRITICAL: Always ask the user before transitioning to the next stage.** Use the AskQuestion tool. Never auto-advance.

## Sub-Skills

For direct invocation when you know which stage you need:

| Sub-Skill | When to Use |
|-----------|-------------|
| `/pmp:brainstorm` | Explore ideas, design approaches, produce design doc |
| `/pmp:plan` | Generate implementation plan from spec, roadmap, or GitHub Issues |
| `/pmp:review` | Skeptical senior-engineer review of an existing plan |
| `/pmp:execute` | Code-test-fix loop, implements plan with agent teams. Also: test-only mode |
| `/pmp:spec-review` | Deep 15-phase architecture & spec analysis (standalone) |
| `/pmp:discuss` | Structured walkthrough of review findings, collect fixes into a plan |
| `/pmp:github` | Publish plan as GitHub Issues/Projects, or sync changes to existing issues |
| `/pmp:decompose` | Break large plans into dependency-ordered phases |
| `/pmp:changelog` | Generate user-facing release notes from completed plans |

## Routing

When the user's intent maps to a specific stage, read the reference for that stage directly:

| Signal | Stage | Reference |
|--------|-------|-----------|
| Idea, feature request, "what if", "design this" | Brainstorm | [brainstorm.md](../brainstorm/references/brainstorm.md) |
| Spec, roadmap, requirements, "create a plan" | Plan | [generate-plans.md](../plan/references/generate-plans.md) |
| GitHub issue URL, epic number, "plan from issues" | Plan (Issues Mode) | [generate-plans.md](../plan/references/generate-plans.md) |
| Existing plan file, "review this" | Review | [review.md](../review/references/review.md) |
| "execute plan", "implement", "start coding" | Execute | [execute-loop.md](../execute/references/execute-loop.md) |
| "review specs", "architecture review", "threat model" | Spec Review | [spec-review.md](../spec-review/references/spec-review.md) |
| "create issues", "make an epic" | GitHub | [github-planning.md](../github/references/github-planning.md) |
| "sync issues", "update issues" | GitHub (Sync) | [sync-issues.md](../github/references/sync-issues.md) |
| "discuss review", "walk through findings", "go through the review" | Discuss | [discuss.md](../discuss/references/discuss.md) |
| "decompose plan", "break into phases", "phase this plan" | Decompose | [decompose.md](../decompose/references/decompose.md) |
| "generate release notes", "changelog", "what was built" | Changelog | [changelog.md](../changelog/references/changelog.md) |
| "run tests", "re-test" | Execute (Test Only) | [execute-loop.md](../execute/references/execute-loop.md) |
| Existing plan + "extend" | Plan (Extend) | [generate-plans.md](../plan/references/generate-plans.md) |

### Lifecycle Flow

Workflows 1–4 share a path: each stage hands off to the next with user confirmation.

1. **Brainstorm** → asks "Ready to generate the plan?" → **Plan**
2. **Plan** → asks "Ready for plan review?" → **Review**
3. **Review** → asks "Publish as GitHub Issues?" → **GitHub** (optional) → **Execute**
4. **Execute** → implements, creates PR, archives plan → optionally **Changelog**

Workflow 5 (**Spec Review**) is standalone — produces a report, no execution.

**Discuss** can follow either Review or Spec Review — walks through findings interactively, collects fixes into a plan, then optionally hands off to Execute.

**Decompose** can be invoked on any existing plan with 5+ features to add phase boundaries. Also auto-triggers during plan generation.

**Changelog** can follow Execute or be invoked standalone on any completed plan.

## Project Rules

Non-negotiable across all modes:

- **Branching:** Branch from detected integration branch, PR back to it. NEVER commit to `main` unless it IS the integration branch
- **Security in every plan:** Input validation, auth boundaries, secret handling, injection risks, attack surface
- **Tests with every change:** Implementation commits include unit/integration tests. E2E tests in separate commit per feature
- **CI gate:** Detected CI command must pass clean before any commit
- **What, not how:** Plans describe behavior, not implementation — the coding agent determines how
- **Context management:** Read [config.md](config.md) before Execute. Batch controllers every 3 features, demand structured returns. Context exhaustion is the #1 cause of execution failures.
- **Verification gate:** Before claiming any mode is complete — run the command, read output, verify. Evidence before assertions.

## Configuration & References

All constants (paths, thresholds, labels, commits, context management) live in [config.md](config.md). Read it before any stage.

For lifecycle diagrams, file tree, assets table, and changelog: [overview.md](references/overview.md).
