# Ragna Claude Plugins

Claude Code plugins for Ragna integration.

## Structure

```
.
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/                # Slash commands
│   └── example.md
├── agents/                  # Specialized agents
│   └── example-agent.md
├── skills/                  # Reusable capabilities
│   └── example-skill/
│       └── SKILL.md
├── hooks/                   # Event handlers
│   └── hooks.json
└── README.md
```

## Development

### Commands

Create markdown files in `commands/` with YAML frontmatter:

```markdown
---
description: Command description
---

Command implementation instructions here.
```

### Agents

Create markdown files in `agents/` that define specialized agent behavior:

```markdown
---
name: agent-name
description: Agent description
---

Agent instructions here.
```

### Skills

Create `SKILL.md` files in subdirectories under `skills/`:

```markdown
---
name: skill-name
description: Skill description
---

Skill implementation instructions here.
```

### Hooks

Define event handlers in `hooks/hooks.json`:

```json
{
  "hooks": {
    "event-name": {
      "command": "shell command",
      "description": "Hook description"
    }
  }
}
```

## Testing

To test this plugin locally:

1. Create a `.claude-plugin/marketplace.json` file
2. Reference this plugin in the marketplace
3. Install using Claude Code

## Installation

Follow the Claude Code plugin installation documentation to install this plugin.

## License

Add your license here.
