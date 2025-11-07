# Ragna Claude Plugins

Claude Code plugins for RagnaRokkrr integration.

## Available Commands

### `/rgn.add-reference`

Add and manage project references (articles, files, URLs, YouTube videos).

**Usage:**
```
# Single reference
/rgn.add-reference <url|file|source>
/rgn.add-reference <url> --mindmap
/rgn.add-reference <url> --tags "tag1,tag2"

# Multiple references
/rgn.add-reference <url1> <url2> <file.pdf>
/rgn.add-reference url1, url2, url3 --mindmap --tags "ai,ml"

# Process local files
/rgn.add-reference --process-local [filename(s)]
```

**Features:**
- Downloads and converts references to markdown
- Processes YouTube videos with transcript extraction
- Generates summaries with automatic tagging
- Creates tab-separated plain-text mindmaps (optional)
- Processes locally managed files from `articles/` directory
- Organizes references in `.memory/references/References.md`
- Maintains full content, summaries, and mindmaps

**Mindmap Format:**
- Plain-text, tab-separated hierarchical structure
- First line contains article title and original URL
- Each indentation level uses one tab character
- Suitable for import into mindmapping tools

**Examples:**
```
# Single references
/rgn.add-reference https://arxiv.org/abs/2301.xxxxx --mindmap
/rgn.add-reference https://www.youtube.com/watch?v=xxxxx
/rgn.add-reference ./papers/important-paper.pdf --tags "ml,rag"

# Multiple references (batch processing)
/rgn.add-reference url1 url2 url3 --mindmap --tags "ai,research"
/rgn.add-reference https://blog.com/post1 ./file.pdf https://youtube.com/watch?v=xyz

# Locally managed files (kept in articles/)
/rgn.add-reference --process-local
/rgn.add-reference --process-local file1.pdf file2.md --tags "research" --mindmap
```

## Structure

```
.
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── .memory/
│   └── references/          # Reference management
│       ├── References.md    # Index of all references
│       ├── articles/        # Manually added files (txt, md, pdf)
│       ├── contents/        # Downloaded and converted content
│       ├── summaries/       # Reference summaries (markdown)
│       └── mindmaps/        # Conceptual mindmaps (tab-separated txt)
├── commands/                # Slash commands
│   ├── example.md
│   └── rgn.add-reference.md
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
