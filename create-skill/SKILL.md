---
name: create-skill
description: Creates Claude Code skills following Anthropic best practices and the official format. Use when the user wants to create, write, or author a new skill for Claude Code, or asks about skill structure, SKILL.md format, or best practices.
argument-hint: [skill-name] [purpose]
---

# Create Claude Code Skills

Guides creation of skills for [Claude Code](https://code.claude.com/docs/en/skills), following [Anthropic's Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf). Skills extend Claude with custom instructions and can be invoked via `/skill-name` or loaded automatically when relevant.

## Invocation

- **Create a new skill**: `/create-skill my-skill-name [brief purpose]`
- **Infer from context**: If we've discussed a workflow, ask whether to turn it into a skill and gather requirements

Arguments: `$0` = skill name (lowercase, hyphens), `$1` = purpose or trigger scenario.

---

## Use Case First

Before writing code, define 2–3 concrete use cases. Format:

```
Use Case: [Name]
Trigger: User says "[phrase]" or "[paraphrase]"
Steps:
1. [Step 1]
2. [Step 2]
Result: [What success looks like]
```

Ask: What does the user want to accomplish? What multi-step workflows? Which tools (built-in or MCP)? What domain knowledge to embed?

---

## Skill Categories (Anthropic)

| Category | Use for | Key techniques |
|----------|---------|----------------|
| **Document & asset creation** | Consistent output (docs, code, designs) | Style guides, templates, quality checklists |
| **Workflow automation** | Multi-step processes | Step-by-step with validation gates, iterative refinement |
| **MCP enhancement** | Guiding MCP tool usage | Sequence MCP calls, embed domain expertise |

---

## Progressive Disclosure

Skills use three levels to minimize tokens:

1. **Frontmatter** (always loaded): Name + description. Enough for Claude to know *when* to use the skill.
2. **SKILL.md body** (when relevant): Full instructions. Loaded when skill appears relevant.
3. **Linked files** (as needed): `references/`, `examples/`, `scripts/`. Claude discovers only when needed.

---

## Before Creating: Gather Requirements

1. **Use cases**: 2–3 concrete scenarios (trigger + steps + result)
2. **Location**: Personal (`~/.claude/skills/`) vs project (`.claude/skills/`)
3. **Invocation**: Auto-load vs manual-only (`disable-model-invocation: true` for side effects)
4. **Arguments**: `$ARGUMENTS`, `$0`, `$1`?
5. **Supporting files**: references/, scripts/, assets/?

---

## Skill Structure (Claude Code)

### Directory layout

```
skill-name/              # kebab-case, matches name field
├── SKILL.md             # Required — exact spelling, case-sensitive
├── references/          # Optional — docs loaded as needed
│   └── api-guide.md
├── scripts/             # Optional — executable code
│   └── validate.sh
├── assets/              # Optional — templates, icons
│   └── report-template.md
└── examples/            # Optional — sample outputs
```

**Critical rules (Anthropic):**

- File must be exactly `SKILL.md` (not SKILL.MD, skill.md)
- Folder: kebab-case only — `my-skill` ✓; no spaces, underscores, or capitals
- **No README.md** inside the skill folder; put docs in SKILL.md or references/

Keep SKILL.md under ~5,000 words; move detailed material to references/ and link to it.

### Where skills live

| Location | Path | Scope |
|----------|------|-------|
| Personal | `~/.claude/skills/<skill-name>/` | All your projects |
| Project | `.claude/skills/<skill-name>/` | This repo only |
| Commands (legacy) | `.claude/commands/<name>.md` | Same as skills; skills directory takes precedence |

---

## Frontmatter Reference

```yaml
---
name: skill-name              # Required. kebab-case, matches folder. Max 64 chars.
description: What it does and when to use it  # Required. Critical for auto-loading.
argument-hint: [filename]     # Optional. Shown in / menu autocomplete
disable-model-invocation: true   # Optional. Only user can invoke (e.g. /deploy)
user-invocable: false        # Optional. Hide from / menu; Claude-only
allowed-tools: Read, Grep     # Optional. Tools Claude can use without asking
context: fork                # Optional. Run in isolated subagent (Claude Code)
agent: Explore               # Optional. When context fork — Explore, Plan, or general-purpose
model: claude-sonnet-4-20250514  # Optional. Model when skill is active
license: MIT                 # Optional. For open-source skills
compatibility: Claude Code, requires gh CLI  # Optional. 1-500 chars. Environment requirements.
metadata:                    # Optional. Custom key-value pairs
  author: YourName
  version: 1.0.0
  mcp-server: server-name
---
```

**Security (Anthropic):** No XML angle brackets (&lt; &gt;) in frontmatter. No "claude" or "anthropic" in skill name (reserved).

**Invocation control**

| Frontmatter | User invokes | Claude invokes | When loaded |
|-------------|--------------|----------------|-------------|
| (default) | ✓ | ✓ | Description in context; full skill when invoked |
| `disable-model-invocation: true` | ✓ | ✗ | Full skill only when you invoke |
| `user-invocable: false` | ✗ | ✓ | Description in context; full skill when Claude invokes |

Use `disable-model-invocation: true` for workflows with side effects (deploy, commit, etc.).

---

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed (e.g. `/fix-issue 123` → `123`) |
| `$ARGUMENTS[0]` or `$0` | First argument |
| `$ARGUMENTS[1]` or `$1` | Second argument |
| `${CLAUDE_SESSION_ID}` | Current session ID (logging, session-specific files) |

---

## Advanced Patterns

### Dynamic context (`!`command\`\`)

Run shell commands **before** Claude sees the skill; output replaces the placeholder:

```yaml
---
name: pr-summary
description: Summarize pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## PR context
- Diff: !`gh pr diff`
- Changed files: !`gh pr diff --name-only`

Summarize this pull request...
```

### Subagent execution

Add `context: fork` when the skill should run in isolation (no conversation history). Use `agent: Explore` for read-only research, `agent: Plan` for planning.

### Bundled scripts

Include scripts in `scripts/` that Claude executes. Document the command and expected output in SKILL.md.

---

## Description Best Practices (Anthropic)

Formula: `[What it does] + [When to use it] + [Key capabilities]`. Under 1024 chars. No XML tags.

1. **Third person**: "Generates commit messages from git diffs" not "I can help you..."
2. **Include WHAT and WHEN**: Specific trigger phrases users would say.
3. **Mention file types** if relevant (e.g. ".fig files", "CSV files").

```yaml
# Good — specific and actionable
description: Generate commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.

# Good — includes triggers
description: Manages Linear project workflows including sprint planning and task creation. Use when user mentions "sprint", "Linear tasks", or "create tickets".

# Vague
description: Helps with git
```

**Over-triggering?** Add negative triggers: "Do NOT use for simple data exploration (use data-viz skill instead)."

---

## Instruction Structure (Anthropic)

Recommended sections in SKILL.md body:

1. **Instructions** — Step-by-step, specific and actionable. Include expected outputs and error handling.
2. **Examples** — Concrete scenarios: "User says X → Actions → Result."
3. **Troubleshooting** — Common errors, causes, solutions.
4. **Additional resources** — Link to references/ files. Keep references one level deep.

Be specific: "Run `python scripts/validate.py --input {filename}`" not "Validate the data." For critical checks, prefer bundled scripts over prose—code is deterministic.

---

## Common Patterns (Anthropic)

- **Sequential workflow**: Explicit steps, dependencies, validation gates, rollback for failures.
- **Iterative refinement**: Initial draft → validation script → fix issues → repeat until quality met.
- **Context-aware tool selection**: Decision tree (e.g. file type/size) → pick tool → execute.
- **Domain-specific intelligence**: Embed expertise (compliance checks, best practices) before action.

---

## Creation Workflow

1. **Define use cases** — 2–3 concrete scenarios before writing.
2. **Create directory**: `mkdir -p <location>/skills/<skill-name>` (kebab-case).
3. **Write SKILL.md**: Frontmatter + Instructions + Examples + Troubleshooting; use [template.md](template.md).
4. **Add supporting files** — references/, scripts/, assets/ as needed.
5. **Verify** — Run through [reference.md](reference.md) checklist.

---

## Troubleshooting (Anthropic)

| Symptom | Fix |
|---------|-----|
| **Skill won't trigger** | Revise description. Add trigger phrases users actually say. Include relevant file types. Avoid vague terms ("helps with projects"). |
| **Skill triggers too often** | Add negative triggers ("Do NOT use for..."). Be more specific. Clarify scope. |
| **Instructions not followed** | Keep instructions concise. Put critical items at top. Use explicit validation ("CRITICAL: Before X, verify..."). For deterministic checks, bundle a script. |
| **Invalid frontmatter** | Check `---` delimiters, closed quotes, YAML syntax. No XML. |
| **Invalid skill name** | kebab-case only; no spaces or capitals. |

---

## Template and reference

- **Starter template**: [template.md](template.md) — fill in name, description, instructions.
- **Full reference**: [reference.md](reference.md) — frontmatter, dynamic context, permissions, checklist.

**Iteration**: Use the skill-creator skill (Claude.ai) or bring edge-case failures back for refinement.

---

## Validation Checklist (before upload)

**During development**

- [ ] Folder and name: kebab-case, match exactly
- [ ] File exactly `SKILL.md` (case-sensitive)
- [ ] YAML frontmatter has `---` delimiters
- [ ] Description includes WHAT and WHEN
- [ ] No XML tags (&lt; &gt;) anywhere
- [ ] Instructions clear and actionable; error handling included
- [ ] Examples provided
- [ ] No README.md in skill folder
- [ ] References linked from SKILL.md

**Triggering**

- [ ] Triggers on obvious tasks
- [ ] Triggers on paraphrased requests
- [ ] Does NOT trigger on unrelated topics

**After upload**

- [ ] Test in real conversations
- [ ] Monitor under/over-triggering; iterate on description
