# Ragna Claude Plugins

Claude Code plugins for RagnaRokkrr integration, providing specialized agents, workflow automation commands, reusable skills, and event-driven hooks.

## Overview

This plugin extends Claude Code with RagnaRokkrr-specific capabilities:
- **Backend architecture design** with comprehensive documentation generation
- **Reference management** for articles, papers, and knowledge bases
- **Event automation** through configurable hooks
- **Reusable skills** for common development tasks

**Version:** 1.0.0

## Features

### Agents

#### backend-architect

Senior backend architect agent that designs scalable service architectures and produces comprehensive arc42 documentation.

**Capabilities:**
- API design (REST, GraphQL, gRPC, WebSocket, event-driven)
- Architecture patterns (Hexagonal, Vertical Slice, DDD, microservices, modular monoliths)
- Complete arc42 documentation generation (sections 1-12)
- Architecture Decision Records (ADRs) with trade-off analysis
- Diagram generation (C4, UML sequence, state machines, ER diagrams)
- Multi-tenancy and security patterns
- Performance, scalability, and observability strategies

**Output:**
- `.spec/architecture/application-architecture.md` - Main arc42 document
- `.spec/architecture/adrs/adr-XXXX.md` - Individual ADR files
- Embedded Mermaid diagrams for all views
- API contract tables (REST endpoints)
- Async API specifications (event channels)
- Quality requirement scorecards
- Risk analysis and mitigation strategies

**When to Use:**
- Designing new backend services or microservices
- Planning API architectures
- Documenting existing systems
- Technical specification for stakeholder review
- Architecture decision documentation

**Example Usage:**
```
Design a REST API backend for a task management system with:
- Multi-tenant data isolation
- Real-time notifications via WebSocket
- External calendar integration (Google, Outlook)
- 10,000 concurrent users
- Small team (3-5 developers)
- 3-month MVP timeline
```

**Agent Properties:**
- Model: inherit
- Color: blue
- Tools: Write, Read, MultiEdit, Bash

---

### Commands

#### `/rgn.add-reference`

Comprehensive reference management system for articles, papers, videos, and documentation.

**Usage:**
```bash
# Single reference
/rgn.add-reference <url|file|source>
/rgn.add-reference <url> --mindmap
/rgn.add-reference <url> --tags "tag1,tag2"

# Multiple references (batch processing)
/rgn.add-reference <url1> <url2> <file.pdf>
/rgn.add-reference url1, url2, url3 --mindmap --tags "ai,ml"

# Process local files
/rgn.add-reference --process-local [filename(s)]
```

**Features:**
- **Multi-source support**: URLs, local files, PDFs, YouTube videos
- **Batch processing**: Accept multiple references in a single command
- **YouTube transcripts**: Extract using yt-dlp or youtube-transcript-api
- **Content conversion**: Download and convert to markdown
- **Auto-summaries**: AI-generated TL;DR (2-3 paragraphs)
- **Metadata extraction**: Author, date, title
- **Mindmap generation**: Tab-separated hierarchical mindmaps (optional `--mindmap` flag)
- **Custom tagging**: User-defined tags (optional `--tags` flag)
- **Local file processing**: Process files from `.memory/references/articles/` directory
- **Organized storage**: Structured in `.memory/references/` with index

**Mindmap Format:**
- Plain-text, tab-separated hierarchical structure
- First line contains article title and original URL
- Each indentation level uses one tab character
- Suitable for import into mindmapping tools (XMind, FreeMind, etc.)

**Examples:**
```bash
# Single references with options
/rgn.add-reference https://arxiv.org/abs/2301.xxxxx --mindmap
/rgn.add-reference https://www.youtube.com/watch?v=xxxxx
/rgn.add-reference ./papers/important-paper.pdf --tags "ml,rag"

# Batch processing multiple sources
/rgn.add-reference https://blog.com/post1 https://blog.com/post2 --mindmap --tags "architecture"
/rgn.add-reference https://example.com/article ./local-file.pdf https://youtube.com/watch?v=xyz

# Process locally managed files (kept in articles/ directory)
/rgn.add-reference --process-local
/rgn.add-reference --process-local paper1.pdf paper2.md --tags "research" --mindmap
```

**Directory Structure:**
```
.memory/references/
├── References.md           # Master index of all references
├── articles/              # Manually added files (txt, md, pdf)
├── contents/              # Downloaded and converted content (markdown)
├── summaries/             # AI-generated summaries (markdown)
└── mindmaps/              # Conceptual mindmaps (tab-separated txt)
```

---

### Skills

#### example-skill

Demonstrates skill structure and organization for creating reusable capabilities.

**Purpose:**
- Template for creating new plugin skills
- Shows proper SKILL.md structure
- Documents required frontmatter

**Usage:**
Invoke during conversations when reusable capability is needed. Skills provide context and instructions that agents can follow.

---

### Hooks

Event-driven automation triggered by Claude Code activities.

**Configured Hooks:**

**`user-prompt-submit`**
- Triggered when user submits a prompt
- Current: Echo notification (example/placeholder)

**`tool-use`**
- Triggered when Claude uses a tool
- Current: Echo tool name (example/placeholder)
- Template variable: `{{tool_name}}`

**Configuration:** `hooks/hooks.json`

**Customization:**
Edit `hooks/hooks.json` to add custom shell commands for automation:
- Code formatting on commits
- Test execution
- Documentation generation
- Validation checks

---

## Installation

### From Marketplace

```bash
# Add marketplace (if not already added)
/plugin marketplace add /home/npadilha/prj/ragna-claude-marketplace

# Install plugin
/plugin install ragna-claude-plugins

# Verify installation
/plugin list
```

### Verify Components

```bash
# Check available commands
/rgn.add-reference --help

# Test backend-architect by asking for architecture design
# (Agents work through conversation, not via Task tool for custom plugins)
```

---

## Plugin Structure

```
ragna-claude-plugins/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest (name, version, author)
├── .memory/
│   └── references/              # Reference management storage
│       ├── References.md        # Master index
│       ├── articles/            # Source files (user-added)
│       ├── contents/            # Converted markdown content
│       ├── summaries/           # AI-generated summaries
│       └── mindmaps/            # Hierarchical concept maps
├── agents/
│   └── backend-architect.md     # Architecture design agent
├── commands/
│   └── rgn.add-reference.md     # Reference management command
├── skills/
│   └── example-skill/
│       └── SKILL.md             # Example reusable skill
├── hooks/
│   └── hooks.json               # Event hook configuration
├── AGENTS.md                    # Agent development guidance
├── CLAUDE.md                    # Claude Code-specific guidance
└── README.md                    # This file
```

---

## Use Cases

### Architecture Design & Documentation

**Scenario:** Design a new microservices backend

1. Describe requirements in conversation:
   ```
   Design a microservices architecture for an e-commerce platform with:
   - Product catalog service
   - Order management service
   - Payment processing service
   - Inventory service
   - Event-driven communication
   - CQRS pattern for read-heavy catalog
   ```

2. backend-architect agent will:
   - Ask clarifying questions about scale, constraints, tech stack
   - Propose architecture with bounded contexts
   - Generate complete arc42 documentation
   - Create ADRs for key decisions (messaging, data consistency, etc.)
   - Produce C4 diagrams, sequence diagrams, ER diagrams
   - Save to `.spec/architecture/`

**Output Example:**
- `application-architecture.md` - 12-section arc42 document
- `adrs/adr-0001.md` - "Microservices vs Monolith"
- `adrs/adr-0002.md` - "Event Sourcing for Orders"
- `adrs/adr-0003.md` - "PostgreSQL for Transactional Data"

### Knowledge Base Management

**Scenario:** Build a research reference library

```bash
# Add academic papers
/rgn.add-reference https://arxiv.org/abs/2301.12345 --tags "rag,llm" --mindmap

# Add blog posts
/rgn.add-reference https://blog.example.com/architecture-patterns --tags "architecture"

# Add YouTube tech talks
/rgn.add-reference https://www.youtube.com/watch?v=xxxxx --tags "conference,microservices"

# Batch add multiple sources
/rgn.add-reference url1 url2 url3 --mindmap --tags "distributed-systems"

# Process locally downloaded PDFs
/rgn.add-reference --process-local paper1.pdf paper2.pdf --tags "research"
```

**Benefits:**
- Centralized reference index in `References.md`
- Full markdown content for searchability
- AI summaries for quick review
- Mindmaps for visual learning
- Tag-based organization

### Project Documentation

**Scenario:** Document existing system architecture

1. Use backend-architect to analyze and document current system
2. Generate ADRs for historical decisions
3. Create architecture diagrams for onboarding
4. Maintain in `.spec/architecture/` directory

---

## Development

### Adding New Agents

1. Create `agents/new-agent-name.md`
2. Add YAML frontmatter:
   ```yaml
   ---
   name: new-agent-name
   description: |
     Brief description of agent purpose
   model: inherit  # or sonnet, opus, haiku
   color: blue     # optional
   tools: Write, Read  # optional
   ---
   ```
3. Document agent capabilities, workflows, output formats
4. Test by following agent instructions in conversation

### Adding New Commands

1. Create `commands/new-command.md` or `commands/namespace.command-name.md`
2. Add YAML frontmatter:
   ```yaml
   ---
   description: Command description for help text
   ---
   ```
3. Write detailed instructions for command execution
4. Test with `/command-name`

### Adding New Skills

1. Create directory `skills/new-skill/`
2. Create `skills/new-skill/SKILL.md`
3. Add YAML frontmatter:
   ```yaml
   ---
   name: new-skill
   description: Skill description
   allowed-tools: ["Tool1", "Tool2"]  # optional
   ---
   ```
4. Document skill purpose, instructions, and usage patterns

### Modifying Hooks

1. Edit `hooks/hooks.json`
2. Add or modify hook entries:
   ```json
   {
     "hooks": {
       "event-name": {
         "command": "shell command to execute",
         "description": "What this hook does"
       }
     }
   }
   ```
3. Use template variables: `{{tool_name}}`, `{{prompt}}`, etc.
4. Test by triggering the event

---

## Testing

### Validate Plugin Structure

```bash
cd ragna-claude-plugins
claude plugin validate .
```

### Test Commands

```bash
# In Claude Code
/rgn.add-reference --help
/rgn.add-reference https://example.com/test-article
```

### Test Agents

Agents work through natural conversation. Ask Claude to design an architecture:
```
Design a REST API for a simple task management system
with PostgreSQL backend and real-time notifications
```

The backend-architect agent's instructions will guide the response.

### Test Hooks

Hooks trigger automatically on events:
- Submit a prompt → `user-prompt-submit` fires
- Use a tool → `tool-use` fires

Check Claude Code logs for hook output.

---

## Configuration

### plugin.json

Located at `.claude-plugin/plugin.json`:

```json
{
  "name": "ragna-claude-plugins",
  "description": "Claude Code plugins for RagnaRokkrr",
  "version": "1.0.0",
  "author": {
    "name": "npadilha"
  }
}
```

Update when:
- Bumping version (use semantic versioning)
- Changing plugin name (requires reinstall)
- Updating description
- Adding author details

---

## Troubleshooting

### Command Not Found

```bash
# Check plugin is installed
/plugin list

# Reinstall if needed
/plugin uninstall ragna-claude-plugins
/plugin install ragna-claude-plugins
```

### Agent Not Working as Expected

- Custom plugin agents work through conversation, not the Task tool's subagent_type
- Ensure agent markdown file has proper frontmatter
- Check that instructions are clear and comprehensive

### Reference Command Failing

- Check network connectivity for URL downloads
- Verify local file paths are correct
- Ensure `.memory/references/` directory exists
- Check for write permissions

### Hook Not Executing

- Verify `hooks/hooks.json` is valid JSON
- Check that shell commands are executable
- Review Claude Code logs for hook errors
- Test command independently in terminal

---

## Architecture Documentation

This plugin includes the backend-architect agent which generates:

**arc42 Documentation** - Industry-standard architecture documentation with:
- 12 numbered sections (Introduction through Glossary)
- Context and scope with stakeholder views
- Solution strategy and building blocks
- Runtime and deployment views
- Cross-cutting concepts and quality requirements
- Risk analysis and technical debt tracking

**Architecture Decision Records (ADRs)** - Separate files for each major decision:
- Status, Context, Decision, Consequences format
- Trade-off analysis with alternatives
- Risk mitigation strategies
- Follow-up actions

**Diagrams** - Embedded Mermaid diagrams:
- C4 Context, Container, Component diagrams
- UML sequence diagrams for key flows
- State machines for lifecycle modeling
- Entity-Relationship diagrams for data

**API Specifications** - Contract-first design:
- REST endpoint tables (Method, Endpoint, Description)
- Async API channel specifications (Channel, Message Types, Purpose)
- Authentication and authorization patterns
- Versioning and compatibility strategies

---

## Contributing

### Workflow

1. Create feature branch: `feature/description` or `agent/agent-name`
2. Develop component following structure guidelines
3. Test locally with marketplace installation
4. Update this README if adding user-facing features
5. Update CLAUDE.md if changing development patterns
6. Commit with descriptive message
7. Include component type: `[agent]`, `[command]`, `[skill]`, `[hook]`

### Commit Message Format

```
[component] Brief description

Detailed explanation of changes:
- What was added/changed
- Why it was necessary
- How to use the new feature

Examples or usage patterns if applicable.
```

### Version Bumping

Follow semantic versioning:
- **MAJOR** (x.0.0): Breaking changes to commands/agents/APIs
- **MINOR** (0.x.0): New features, new agents/commands
- **PATCH** (0.0.x): Bug fixes, documentation updates

Update `plugin.json` version field when releasing.

---

## Resources

- [Claude Code Documentation](https://code.claude.com/docs)
- [Plugin Reference](https://code.claude.com/docs/en/plugins-reference.md)
- [Marketplace Guide](https://code.claude.com/docs/en/plugin-marketplaces.md)
- [Skills Guide](https://code.claude.com/docs/en/skills.md)
- [Hooks Guide](https://code.claude.com/docs/en/hooks-guide.md)

## License

Proprietary - RagnaRokkrr

---

**Maintainer:** npadilha
**Last Updated:** 2025-11-10
**Version:** 1.0.0
