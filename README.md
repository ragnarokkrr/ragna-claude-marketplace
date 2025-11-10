# RagnaRokkrr Claude Code Marketplace

Local marketplace for Claude Code plugins tailored to RagnaRokkrr integration and development workflows.

## Overview

This marketplace provides specialized Claude Code plugins that extend Claude's capabilities with:
- Domain-specific agents (backend architecture, database design, etc.)
- Workflow automation commands
- Reusable skills for common tasks
- Event-driven hooks for enhanced productivity

## Installation

### Install the Marketplace

```bash
# In Claude Code CLI
/plugin marketplace add /home/npadilha/prj/ragna-claude-marketplace

# List available plugins
/plugin marketplace list

# Install a plugin
/plugin install ragna-claude-plugins
```

### Verify Installation

```bash
# Check installed plugins
/plugin list

# Show plugin details
/plugin show ragna-claude-plugins
```

## Available Plugins

### ragna-claude-plugins

**Version:** 1.0.0

Core plugin for RagnaRokkrr development workflows.

#### Agents

**backend-architect**
- Designs scalable backend architectures (REST, GraphQL, gRPC, event-driven)
- Generates comprehensive arc42 documentation with diagrams
- Creates Architecture Decision Records (ADRs)
- Supports Hexagonal, Vertical Slice, DDD patterns
- Outputs: API contracts, C4 diagrams, UML sequences, ER diagrams

*Example usage:*
```
Design a microservices architecture for a multi-tenant SaaS platform
with real-time collaboration features
```

#### Commands

**`/rgn.add-reference`**
- Add and manage project references (articles, files, URLs, videos)
- Downloads and converts content to markdown
- Generates AI summaries and mindmaps
- Supports batch processing with tags
- Organizes content in `.memory/references/` structure

*Example usage:*
```bash
/rgn.add-reference https://example.com/article.html --mindmap --tags "architecture,patterns"
/rgn.add-reference ./paper.pdf https://video.com --tags "research"
/rgn.add-reference --process-local
```

#### Skills

**example-skill**
- Demonstrates skill structure and organization
- Template for creating new reusable skills

#### Hooks

- `user-prompt-submit`: Triggered when user submits a prompt
- `tool-use`: Triggered when Claude uses a tool

## Quick Start

1. **Install the marketplace:**
   ```bash
   /plugin marketplace add /home/npadilha/prj/ragna-claude-marketplace
   ```

2. **Install the plugin:**
   ```bash
   /plugin install ragna-claude-plugins
   ```

3. **Try the backend architect:**
   ```
   Design a REST API for a task management system with
   PostgreSQL backend, Redis caching, and real-time notifications
   ```

4. **Add a reference:**
   ```bash
   /rgn.add-reference https://martinfowler.com/articles/patterns-of-distributed-systems/ --tags "distributed-systems"
   ```

## Plugin Structure

```
ragna-claude-plugins/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
├── agents/
│   └── backend-architect.md  # Architecture design agent
├── commands/
│   └── rgn.add-reference.md  # Reference management command
├── skills/
│   └── example-skill/
│       └── SKILL.md
├── hooks/
│   └── hooks.json           # Event hooks
├── .memory/                 # Reference storage
│   └── references/
│       ├── articles/        # Source files
│       ├── contents/        # Converted content
│       ├── summaries/       # AI summaries
│       └── mindmaps/        # Conceptual maps
├── CLAUDE.md               # Developer guidance
└── README.md               # Plugin documentation
```

## Use Cases

### Architecture Design

Use the `backend-architect` agent to:
- Design new services from scratch
- Modernize legacy systems
- Plan microservices decomposition
- Document existing architectures
- Create technical specifications for teams

### Knowledge Management

Use `/rgn.add-reference` to:
- Collect technical articles for research
- Archive important documentation
- Build a personal knowledge base
- Generate summaries for quick review
- Create mindmaps for complex topics

### Project Setup

- Initialize architecture documentation structure
- Set up reference libraries for new projects
- Create ADR templates and tracking
- Establish documentation conventions

## Advanced Usage

### Custom Agents

Create specialized agents for:
- Frontend architecture (React, Vue, Angular)
- Database design (PostgreSQL, MongoDB)
- DevOps pipelines (CI/CD, IaC)
- Security architecture
- Performance optimization

### Command Composition

Chain commands for complex workflows:
```bash
# Fetch references, process, and generate architecture
/rgn.add-reference <urls> --tags "requirements"
# Then use backend-architect to design system
```

### Hook Automation

Configure hooks for:
- Auto-formatting code on commits
- Running tests before deployments
- Generating documentation on changes
- Validating architecture decisions

## Development

### Adding New Agents

1. Create agent markdown file in `agents/`
2. Add YAML frontmatter with name and description
3. Document capabilities, workflows, and examples
4. Test with Task tool

### Adding New Commands

1. Create command markdown file in `commands/`
2. Add description in YAML frontmatter
3. Write detailed instructions for execution
4. Test with `/command-name`

### Modifying Hooks

1. Edit `hooks/hooks.json`
2. Add event type and command
3. Use template variables (e.g., `{{tool_name}}`)
4. Test by triggering events

## Architecture Documentation

The marketplace includes sample architecture documentation in `.spec/architecture/`:

- **application-architecture.md**: Complete arc42 documentation for a task management system
- **ADRs**: Five architecture decision records covering key technical choices

These serve as:
- Examples of backend-architect agent output
- Templates for your own projects
- Reference for documentation standards

## Troubleshooting

### Plugin Not Loading

```bash
# Validate plugin structure
cd ragna-claude-plugins
claude plugin validate .

# Reinstall plugin
/plugin uninstall ragna-claude-plugins
/plugin install ragna-claude-plugins
```

### Command Not Found

```bash
# Check plugin is installed
/plugin list

# Verify command exists
ls ragna-claude-plugins/commands/
```

### Agent Not Available

Custom agents defined in plugins work differently than built-in Task tool agents. Follow agent instructions directly in conversations rather than using the Task tool's subagent_type parameter.

## Roadmap

Future plugin additions:
- **database-architect**: PostgreSQL schema design and optimization
- **frontend-architect**: React/Vue/Angular architecture patterns
- **cloud-architect**: AWS/GCP/Azure infrastructure design
- **security-auditor**: Security analysis and recommendations
- **performance-engineer**: Performance optimization strategies

## Contributing

### Adding to This Marketplace

1. Create new plugin directory
2. Add plugin.json with manifest
3. Update marketplace.json
4. Test locally
5. Commit and document

### Best Practices

- Follow Claude Code plugin specifications
- Use descriptive names and documentation
- Test thoroughly before committing
- Version plugins semantically
- Document breaking changes

## Support

For issues or questions:
- Check [Claude Code Documentation](https://code.claude.com/docs)
- Review `CLAUDE.md` for development guidance
- Examine example implementations in existing plugins

## License

This marketplace and its plugins are proprietary to RagnaRokkrr.

---

**Version:** 1.0.0
**Last Updated:** 2025-11-10
**Maintainer:** npadilha
