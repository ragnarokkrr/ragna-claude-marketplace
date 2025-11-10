# CLAUDE.md

This file provides guidance to Claude Code when working with this plugin codebase.

## Project Overview

This is the **ragna-claude-plugins** plugin for Claude Code, providing RagnaRokkrr-specific capabilities including backend architecture design, reference management, and development workflow automation.

**Plugin Type:** Claude Code Plugin (commands, agents, skills, hooks)
**Target Integration:** RagnaRokkrr development workflows
**Version:** 1.0.0

## Component Inventory

### Agents (`agents/`)

**backend-architect.md** (11,603 lines)
- Senior backend architect agent for scalable service design
- Generates arc42 documentation with C4, UML, and ER diagrams
- Creates Architecture Decision Records (ADRs)
- Supports Hexagonal, Vertical Slice, DDD, microservices patterns
- Output: `.spec/architecture/application-architecture.md` and `adrs/`

### Commands (`commands/`)

**rgn.add-reference.md** (13,796 lines)
- Reference management system for articles, papers, videos, docs
- Multi-source support (URLs, files, PDFs, YouTube)
- Batch processing with tags and mindmap generation
- Stores in `.memory/references/` with organized structure

### Skills (`skills/`)

**example-skill/** - Template demonstrating skill structure

### Hooks (`hooks/hooks.json`)

- `user-prompt-submit` - Triggered on prompt submission
- `tool-use` - Triggered when tools are used

## Plugin Architecture

### Manifest System

**`.claude-plugin/plugin.json`** - Core metadata:
```json
{
  "name": "ragna-claude-plugins",
  "description": "Claude Code plugins for RagnaRokkrr",
  "version": "1.0.0",
  "author": {"name": "npadilha"}
}
```

**Critical:** Any changes to name, version, description, or author must be updated here. This file is read during plugin installation and loading.

### Component Types & Structure

#### Commands (Slash Commands)
- **Location:** `commands/*.md`
- **Naming:** `command-name.md` or `namespace.command-name.md`
- **Invocation:** `/command-name`
- **Frontmatter:** YAML with `description` field
- **Content:** Instructions that expand into conversation context

Example structure:
```markdown
---
description: Brief command description
---

Detailed instructions for executing the command...
```

#### Agents (Specialized Behaviors)
- **Location:** `agents/*.md`
- **Naming:** `agent-name.md` (kebab-case)
- **Usage:** Follow agent instructions in conversation (not via Task tool subagent_type)
- **Frontmatter:** YAML with `name`, `description`, optional `model`, `color`, `tools`
- **Content:** Purpose, capabilities, workflows, output formats, examples

Example structure:
```markdown
---
name: agent-name
description: |
  Multi-line agent description
model: inherit  # or sonnet, opus, haiku
color: blue     # optional
tools: Write, Read, Bash  # optional
---

# Agent Name
Detailed instructions, workflows, templates...
```

#### Skills (Reusable Capabilities)
- **Location:** `skills/skill-name/SKILL.md`
- **Directory:** Each skill in its own subdirectory
- **File:** Must be named `SKILL.md` (uppercase)
- **Frontmatter:** YAML with `name`, `description`, optional `allowed-tools`
- **Invocation:** Used during conversations when capability is needed

Example structure:
```markdown
---
name: skill-name
description: Skill description
allowed-tools: ["Tool1", "Tool2"]  # optional
---

# Skill Name
Purpose, instructions, usage patterns...
```

#### Hooks (Event-Driven Automation)
- **Location:** `hooks/hooks.json`
- **Format:** JSON configuration
- **Events:** `user-prompt-submit`, `tool-use`, `session-start`, etc.
- **Template Variables:** `{{tool_name}}`, `{{prompt}}`, etc.

Example structure:
```json
{
  "hooks": {
    "event-name": {
      "command": "shell command with {{variables}}",
      "description": "What this hook does"
    }
  }
}
```

## Development Workflows

### Adding New Agents

1. **Create agent file:**
   ```bash
   touch agents/new-agent-name.md
   ```

2. **Add frontmatter with required fields:**
   ```yaml
   ---
   name: new-agent-name
   description: |
     Multi-line description of agent purpose and capabilities
   model: inherit
   color: blue
   tools: Write, Read, Bash
   ---
   ```

3. **Document comprehensively:**
   - Purpose and when to use
   - Core capabilities
   - Operating principles
   - Workflow steps
   - Output formats and file locations
   - Templates and examples
   - Quality criteria

4. **Test in conversation:**
   - Agents work through following their instructions
   - NOT via Task tool's subagent_type (that's for built-in agents only)
   - Ask Claude to follow the agent's documented workflow

### Adding New Commands

1. **Create command file:**
   ```bash
   touch commands/namespace.command-name.md
   ```

2. **Add frontmatter:**
   ```yaml
   ---
   description: Command description shown in help
   ---
   ```

3. **Write detailed instructions:**
   - Parameter parsing and validation
   - Step-by-step workflow
   - Error handling
   - Output format
   - Examples

4. **Test command:**
   ```bash
   /namespace.command-name [args]
   ```

### Adding New Skills

1. **Create skill directory and file:**
   ```bash
   mkdir -p skills/new-skill
   touch skills/new-skill/SKILL.md
   ```

2. **Add frontmatter:**
   ```yaml
   ---
   name: new-skill
   description: What this skill does
   allowed-tools: ["Write", "Read"]  # optional restriction
   ---
   ```

3. **Document skill:**
   - Clear purpose
   - When to invoke
   - Instructions for use
   - Expected inputs/outputs

4. **Test invocation:**
   - Invoke naturally in conversation
   - Verify skill context is loaded

### Modifying Hooks

1. **Edit `hooks/hooks.json`:**
   ```json
   {
     "hooks": {
       "new-event": {
         "command": "echo 'Event triggered'",
         "description": "Responds to new event"
       }
     }
   }
   ```

2. **Use template variables:**
   - `{{tool_name}}` - Name of tool being used
   - `{{prompt}}` - User's prompt text
   - Other variables per event type

3. **Test by triggering events:**
   - Submit prompts → `user-prompt-submit`
   - Use tools → `tool-use`
   - Check Claude Code logs for output

## File Management

### Reference Storage (`.memory/references/`)

Managed by `/rgn.add-reference` command:

```
.memory/references/
├── References.md           # Master index (DO NOT manually edit)
├── articles/              # Manually added source files
│   ├── paper.pdf
│   └── notes.md
├── contents/              # Downloaded/converted content (auto-managed)
│   └── article-title.md
├── summaries/             # AI-generated summaries (auto-managed)
│   └── article-title-summary.md
└── mindmaps/              # Concept maps (auto-managed)
    └── article-title-mindmap.txt
```

**Important:**
- Only add files directly to `articles/` directory
- Other directories are auto-managed by the command
- `References.md` is auto-generated index
- Use `--process-local` to process files in `articles/`

### Architecture Documentation (`.spec/architecture/`)

Generated by `backend-architect` agent:

```
.spec/architecture/
├── application-architecture.md  # Main arc42 document
└── adrs/                        # Architecture Decision Records
    ├── adr-0001.md
    ├── adr-0002.md
    └── ...
```

**Important:**
- Generated by agent, not manually created (usually)
- Can be edited after generation for refinement
- ADRs numbered sequentially (adr-0001, adr-0002, etc.)
- Follow arc42 template structure

## Testing & Validation

### Validate Plugin Structure

```bash
cd /home/npadilha/prj/ragna-claude-marketplace/ragna-claude-plugins
claude plugin validate .
```

Checks:
- JSON syntax in `plugin.json` and `hooks.json`
- Required frontmatter in agents, commands, skills
- File naming conventions
- Directory structure compliance

### Test Installation

```bash
# Add marketplace
/plugin marketplace add /home/npadilha/prj/ragna-claude-marketplace

# Install plugin
/plugin install ragna-claude-plugins

# Verify
/plugin list
/plugin show ragna-claude-plugins
```

### Test Commands

```bash
# Invoke command
/rgn.add-reference https://example.com/article

# Check output
ls -la .memory/references/
cat .memory/references/References.md
```

### Test Agents

Ask Claude to follow agent instructions:
```
Design a REST API for a task management system with
PostgreSQL, Redis, and real-time WebSocket notifications
```

The backend-architect agent's instructions will guide the response and generate documentation in `.spec/architecture/`.

### Test Hooks

Trigger events and check Claude Code logs:
- Submit a prompt → Check for `user-prompt-submit` output
- Use a tool → Check for `tool-use` output with tool name

## Version Management

### Semantic Versioning

Follow semver (MAJOR.MINOR.PATCH):

- **MAJOR (x.0.0):** Breaking changes
  - Command signature changes
  - Agent output format changes
  - Directory structure changes
  - Incompatible plugin.json changes

- **MINOR (0.x.0):** New features
  - New agents
  - New commands
  - New skills
  - New hooks
  - Backward-compatible enhancements

- **PATCH (0.0.x):** Bug fixes
  - Agent instruction corrections
  - Command bug fixes
  - Documentation updates
  - Hook fixes

### Updating Version

1. Edit `.claude-plugin/plugin.json`:
   ```json
   {
     "version": "1.1.0"
   }
   ```

2. Update documentation (README.md, etc.)

3. Commit with version tag:
   ```bash
   git add .
   git commit -m "[release] Version 1.1.0 - Description"
   git tag v1.1.0
   ```

## Best Practices

### Agent Development

1. **Be Comprehensive:** Include all necessary context in agent instructions
2. **Specify Outputs:** Define exact file paths, formats, and structures
3. **Provide Templates:** Include templates and examples in agent markdown
4. **Document Decisions:** Explain why certain patterns are used
5. **Test Thoroughly:** Ensure agents produce consistent, high-quality output

### Command Development

1. **Parse Carefully:** Handle all argument combinations and edge cases
2. **Validate Inputs:** Check for required parameters and valid values
3. **Error Gracefully:** Provide clear error messages with guidance
4. **Document Examples:** Show multiple usage patterns
5. **Maintain State:** Update indexes and metadata consistently

### Skill Development

1. **Single Responsibility:** Each skill should do one thing well
2. **Clear Invocation:** Document when and how to use
3. **Tool Restrictions:** Use `allowed-tools` to limit scope when needed
4. **Self-Contained:** Skills should work independently
5. **Composable:** Design skills to work together when appropriate

### Hook Development

1. **Non-Blocking:** Keep hooks fast and lightweight
2. **Idempotent:** Hooks may fire multiple times
3. **Defensive:** Handle errors without breaking workflow
4. **Informative:** Use descriptive echo messages for debugging
5. **Secure:** Validate inputs, avoid destructive operations

## Troubleshooting

### Agent Not Following Instructions

**Issue:** Agent doesn't produce expected output

**Diagnosis:**
1. Check agent frontmatter is valid YAML
2. Verify `name` and `description` fields exist
3. Ensure instructions are clear and unambiguous
4. Confirm file is in `agents/` directory

**Fix:**
- Rewrite instructions more explicitly
- Add more examples and templates
- Break complex workflows into numbered steps

### Command Not Found

**Issue:** `/command-name` returns "command not found"

**Diagnosis:**
1. Check file exists in `commands/` directory
2. Verify frontmatter has `description` field
3. Confirm plugin is installed (`/plugin list`)
4. Check file naming matches command invocation

**Fix:**
- Reinstall plugin: `/plugin uninstall` then `/plugin install`
- Verify file name matches command (underscores become hyphens)

### Reference Command Failures

**Issue:** `/rgn.add-reference` fails to process references

**Diagnosis:**
1. Check network connectivity for URL downloads
2. Verify `.memory/references/` directory exists
3. Check write permissions
4. Review error messages for specifics

**Fix:**
- Create directory: `mkdir -p .memory/references/{articles,contents,summaries,mindmaps}`
- Fix permissions: `chmod -R u+w .memory/`
- Check URL accessibility in browser

### Hook Not Executing

**Issue:** Hook doesn't trigger on events

**Diagnosis:**
1. Verify `hooks/hooks.json` is valid JSON
2. Check event name spelling
3. Review Claude Code logs for errors
4. Test command independently in terminal

**Fix:**
- Validate JSON: `jq . hooks/hooks.json`
- Ensure shell commands are executable
- Use absolute paths in commands
- Add error handling with `|| true`

## RagnaRokkrr Integration Context

This plugin is specifically designed for RagnaRokkrr development:

- **Architecture Focus:** Backend services, APIs, microservices
- **Documentation Standards:** arc42, ADRs, C4 diagrams
- **Technology Alignment:** Patterns and tools used in RagnaRokkrr stack
- **Workflow Integration:** Reference management for research and learning
- **Team Collaboration:** Shared documentation and knowledge base

When developing new components:
- Align with RagnaRokkrr's technical stack
- Use domain-specific terminology
- Reference RagnaRokkrr patterns and conventions
- Support team workflows and collaboration needs

## Related Documentation

**Plugin-Level:**
- `README.md` - User-facing documentation
- `AGENTS.md` - Agent development guidance
- Individual agent/command files - Component-specific docs

**Marketplace-Level:**
- `../CLAUDE.md` - Marketplace development guidance
- `../README.md` - Marketplace user documentation
- `../.claude-plugin/marketplace.json` - Marketplace manifest

**External Resources:**
- [Claude Code Docs](https://code.claude.com/docs)
- [Plugin Reference](https://code.claude.com/docs/en/plugins-reference.md)
- [Marketplace Guide](https://code.claude.com/docs/en/plugin-marketplaces.md)

## Quick Reference

### File Locations

- Plugin manifest: `.claude-plugin/plugin.json`
- Agents: `agents/*.md`
- Commands: `commands/*.md`
- Skills: `skills/*/SKILL.md`
- Hooks: `hooks/hooks.json`
- References: `.memory/references/`
- Architecture: `.spec/architecture/`

### Common Commands

```bash
# Validate plugin
claude plugin validate .

# Install/reinstall
/plugin install ragna-claude-plugins
/plugin uninstall ragna-claude-plugins

# Test commands
/rgn.add-reference <url>

# Check plugin status
/plugin list
/plugin show ragna-claude-plugins
```

### Editing Checklist

When modifying plugin components:
- [ ] Update version in `plugin.json` (if needed)
- [ ] Update README.md (if user-facing change)
- [ ] Test with `claude plugin validate .`
- [ ] Test installation and functionality
- [ ] Commit with descriptive message including component type
- [ ] Tag release if version bumped

---

**Last Updated:** 2025-11-10
**Plugin Version:** 1.0.0
**Maintained By:** npadilha
