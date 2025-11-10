# AGENTS.md

This document converts `CLAUDE.md` into direct instructions for autonomous coding agents that work inside `ragna-claude-plugins`.

## Mission
- Ship a Claude Code plugin that augments RagnaRokkrr workflows via custom commands, agents, skills, and hooks.
- Keep `.claude-plugin/plugin.json` authoritative; update it whenever metadata changes and bump the semantic version.

## Component Rules
**Commands** (`commands/*.md`)
- Must start with YAML frontmatter containing `description`.
- File name = slash command (`rgn.add-reference.md` → `/rgn.add-reference`).
- Body is injected into the conversation; keep it procedural and deterministic.

**Agents** (`agents/*.md`)
- Begin with YAML frontmatter containing `name`, `description`, and optional `model`, `color`, `tools`.
- Provide clear “When to Use”, “Operating Principles”, and workflow sections so Claude can act autonomously.

**Skills** (`skills/*/SKILL.md`)
- Each skill lives in its own directory.
- Frontmatter must declare `name` and `description`.
- Document invocation steps and expected outcomes.

**Hooks** (`hooks/hooks.json`)
- JSON schema: `{"hooks": {"event-name": {"command": "...", "description": "..."}}}`.
- Supported events include `user-prompt-submit`, `tool-use`, etc.
- Commands should be idempotent bash snippets; note any template variables (e.g., `{{tool_name}}`).

## Development Workflow
1. **Choose component type** using the rules above.
2. **Scaffold files** with correct frontmatter and section headings.
3. **Implement logic** tailored to RagnaRokkrr processes (replace placeholders ASAP).
4. **Test locally** by installing through the marketplace (`/plugin marketplace add …` then `/plugin install ragna-claude-plugins`).
5. **Document changes** in README/CLAUDE if they impact users or future agents.

## Testing & Validation
- Ensure `.memory/references/` structure exists when working on `/rgn.add-reference`.
- Use `jq` to validate JSON manifests before committing.
- After adding commands/agents/skills, reinstall the plugin and trigger each component (slash command, Task tool, skill invocation).

## RagnaRokkrr Focus
Every artifact should support RagnaRokkrr integration scenarios. When adding examples or templates, tailor them to real RagnaRokkrr workflows (architecture documentation, reference gathering, etc.) instead of generic placeholders.
