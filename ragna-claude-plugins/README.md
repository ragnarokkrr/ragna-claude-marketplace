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

#### principal-engineer

Principal engineer agent that scaffolds production-ready Java Spring Boot projects from architecture specifications created by backend-architect.

**Capabilities:**
- Architecture document parsing (arc42 format, ADRs)
- Maven/Gradle build configuration generation with JDK 24+ support
- Package structure generation following architectural patterns (Hexagonal, Vertical Slice, DDD, Layered)
- Skeleton code generation (controllers, services, repositories, entities, DTOs, configuration classes)
- Multi-module project setup for microservices
- Docker and Kubernetes manifest generation
- Comprehensive handoff documentation (README, IMPLEMENTATION-GUIDE)
- Integration with spring-boot-engineer for implementation

**Output:**
- Complete Maven/Gradle project structure
- Skeleton Java classes with TODOs for implementation
- Configuration files (application.yml with profiles)
- Docker and Kubernetes manifests
- README.md and IMPLEMENTATION-GUIDE.md
- Test directory structure and base classes

**When to Use:**
- After backend-architect creates architecture documentation
- Starting new Java Spring Boot microservices
- Converting architecture designs into code scaffolding
- Setting up multi-module Maven/Gradle projects
- Preparing projects for team implementation

**Example Usage:**
```
Scaffold a Spring Boot project based on the architecture in .spec/architecture/application-architecture.md:
- Read the arc42 documentation
- Follow the Hexagonal architecture pattern
- Create Maven project structure
- Generate skeleton classes for all components
- Setup Docker and Kubernetes configs
- Prepare handoff documentation
```

**Agent Properties:**
- Model: inherit
- Color: purple
- Tools: Read, Write, Edit, Bash, Glob, Grep, Skill

**Workflow Integration:**
```
backend-architect → principal-engineer → spring-boot-engineer
(Design)          → (Scaffold)         → (Implement)
```

---

#### spring-boot-engineer

Spring Boot 3+ engineer agent that implements production-ready applications with JDK 24+ modern Java features.

**Capabilities:**
- JDK 24+ features (virtual threads, structured concurrency, pattern matching, records, sealed types)
- Spring Boot 3+ application implementation
- Microservices architecture and Spring Cloud integration
- Reactive programming with WebFlux (when appropriate)
- Security implementation (OAuth2, JWT)
- Testing strategies (unit, integration, contract tests with >85% coverage)
- Performance optimization (caching, connection pooling, async processing)
- Production observability (metrics, tracing, logging)

**When to Use:**
- Implementing business logic after project scaffolding
- Adding Spring Boot features to existing projects
- Integrating Spring Cloud components
- Performance tuning and optimization
- Adding comprehensive test coverage

**Agent Properties:**
- Model: inherit
- Color: green
- Tools: Read, Write, Edit, Bash, Glob, Grep, Skill

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

The plugin provides reusable skills organized into two categories:

#### Architecture Skills

Skills for architecture documentation and design (used by backend-architect):

- **create-adr** - Generate Architecture Decision Records
- **c4-diagram** - Generate C4 architecture diagrams (Context, Container, Component)
- **api-draft** - Document REST, GraphQL, gRPC, and async APIs
- **sql-schema** - Document relational database schemas with ER diagrams and DDL
- **mongodb-schema** - Document MongoDB collection schemas
- **sequence-diagram** - Generate UML sequence diagrams
- **state-diagram** - Generate UML state diagrams
- **modules-diagram** - Generate package/module structure diagrams
- **arch-checklist** - Review architecture documentation quality

#### Engineering Skills

Skills for Java Spring Boot project scaffolding and implementation (used by principal-engineer and spring-boot-engineer):

**openapi-from-architecture** - Generate OpenAPI 3.x YAML from architecture docs
- Converts API documentation from arc42 to OpenAPI specification
- Creates contract-first API definitions
- Includes schemas, security, validation, pagination patterns
- Output: `openapi.yaml` ready for controller generation

**controllers-from-openapi** - Generate Spring Boot controllers and DTOs from OpenAPI
- Reads OpenAPI YAML specification
- Generates Request/Response DTOs as Java Records with validation
- Generates REST Controllers with Spring and OpenAPI annotations
- Creates test skeletons with MockMvc
- Follows architecture pattern for package structure

**spring-config** - Generate Spring Boot configuration files
- application.yml with multiple profiles
- Database, security, cache, messaging configuration

**spring-docker** - Generate Docker and Kubernetes manifests
- Multi-stage Dockerfile, docker-compose.yml
- Kubernetes Deployment, Service, ConfigMap manifests

**spring-observability** - Configure observability stack
- Metrics, tracing, logging, custom KPIs

**spring-testing-setup** - Setup comprehensive testing framework
- JUnit 5, Testcontainers, MockMvc, contract tests

**spring-virtual-threads** - Configure JDK 24+ virtual threads
- Virtual thread configuration and optimization

**Usage:**
Skills are invoked automatically by agents during their workflows. For example, principal-engineer uses `openapi-from-architecture` and `controllers-from-openapi` to implement contract-first API development.

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
│   ├── backend-architect.md     # Architecture design agent
│   ├── principal-engineer.md    # Java project scaffolding agent
│   ├── spring-boot-engineer.md  # Spring Boot implementation agent
│   ├── kubernetes-engineer.md   # Kubernetes deployment specialist
│   ├── mongodb-pro.md           # MongoDB expert
│   └── sql-pro.md               # SQL expert
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

### Project Scaffolding from Architecture

**Scenario:** Scaffold Spring Boot project from architecture documentation

1. After backend-architect creates architecture, use principal-engineer:
   ```
   Scaffold a Java Spring Boot project based on the architecture in .spec/architecture/:
   - Read the arc42 documentation and ADRs
   - Follow the Hexagonal architecture pattern
   - Create Maven project with JDK 24+
   - Generate skeleton classes for order-service
   - Setup PostgreSQL and Kafka integration
   - Prepare Docker and Kubernetes manifests
   ```

2. principal-engineer agent will:
   - Parse arc42 documentation for components and requirements
   - Read ADRs for technology choices and patterns
   - Generate Maven/Gradle build configuration
   - Create package structure following Hexagonal architecture
   - Generate skeleton controllers, services, repositories, entities
   - Create application.yml with profiles (dev, test, prod)
   - Generate Docker and Kubernetes manifests
   - Create README.md and IMPLEMENTATION-GUIDE.md

3. spring-boot-engineer agent can then:
   - Implement business logic in service classes
   - Add security configuration (OAuth2/JWT)
   - Write comprehensive tests (>85% coverage)
   - Configure observability (metrics, tracing, logging)
   - Optimize performance with virtual threads

**Output Example:**
- Complete Maven project structure with dependencies
- Package hierarchy: `application/`, `domain/`, `infrastructure/`
- 15+ skeleton Java files with TODOs
- Configuration files with environment-specific profiles
- Docker multi-stage build and docker-compose.yml
- Kubernetes Deployment, Service, and Ingress manifests
- Comprehensive handoff documentation

**Workflow:**
```
backend-architect     → principal-engineer      → spring-boot-engineer
(.spec/architecture/) → (scaffolded project)    → (implemented code)
```

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
