# CLAUDE.md

## Version Bumping

When committing and pushing changes to a plugin, bump the patch version in that plugin's `.claude-plugin/plugin.json` file. For example, `1.8.0` → `1.8.1`. Only bump once per commit, even if multiple files in the same plugin changed.

## Skill File Size Limit

Keep all skill markdown files (SKILL.md, references/*.md, assets/*.md) at or below 500 lines. If a file exceeds 500 lines, split it into multiple focused files in the same directory.
