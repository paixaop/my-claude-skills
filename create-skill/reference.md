# Claude Code Skills — Reference

Full reference for creating skills. See [SKILL.md](SKILL.md) for the creation workflow.

Based on [Anthropic's Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf).

## Frontmatter fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Slash command. kebab-case, matches folder, max 64 chars. No "claude" or "anthropic". |
| `description` | Yes | What + when to use. Under 1024 chars. No XML (&lt; &gt;). |
| `argument-hint` | No | Shown in `/` autocomplete |
| `disable-model-invocation` | No | If true, only user can invoke |
| `user-invocable` | No | If false, hidden from `/` menu |
| `allowed-tools` | No | Tools Claude can use without permission |
| `model` | No | Model when skill is active |
| `context` | No | Set to `fork` for subagent (Claude Code) |
| `agent` | No | When fork: Explore, Plan, or custom |
| `hooks` | No | Lifecycle hooks |
| `license` | No | MIT, Apache-2.0 for open-source |
| `compatibility` | No | 1-500 chars. Environment requirements. |
| `metadata` | No | author, version, mcp-server, etc. |

**Security (Anthropic):** No XML angle brackets. No reserved names.

## Dynamic context syntax

`` !`command` `` runs before Claude sees the skill. Output replaces the placeholder:

```markdown
Current branch: !`git branch --show-current`
Recent commits: !`git log -3 --oneline`
```

## Permissions

Control skill access via `/permissions`:

```
# Deny all skills
Skill

# Allow specific skills
Skill(commit)
Skill(review-pr *)

# Deny specific skills
Skill(deploy *)
```

## Skills vs commands

- **Skills** (`.claude/skills/<name>/SKILL.md`): Directory, supporting files, frontmatter, recommended
- **Commands** (`.claude/commands/<name>.md`): Single file, same frontmatter, legacy

Both create `/name`. If both exist, the skill wins.

## Directory structure (Anthropic)

```
skill-name/
├── SKILL.md       # Required
├── references/    # Docs loaded as needed
├── scripts/       # Executable code
├── assets/        # Templates, icons
└── examples/      # Sample outputs
```

No README.md inside the skill folder.

## Docs

- [Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [Anthropic Complete Guide (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
- [Subagents](https://code.claude.com/en/sub-agents)
- [Permissions](https://code.claude.com/en/permissions)
