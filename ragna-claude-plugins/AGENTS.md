# AGENTS.md

This file provides guidance to any automated coding agent working in this repository (e.g., Claude Code, CodeLlama, GPT-based tools). Use it as the source of truth for expectations and project context.

## Project Overview

This is a Claude Code plugin project for RagnaRokkrr integration. The plugin extends Claude's capabilities through custom commands, agents, skills, and event hooks that the IDE can surface as slash commands or autonomous helpers.

## Architecture

### Plugin Manifest System

The `.claude-plugin/plugin.json` file is the core metadata descriptor. Any changes to plugin name, description, version, or author information must be reflected here. This file is read by Claude Code (and any compatible loader) during plugin installation and loading.

### Component Types

**Commands** (`commands/`)
- Markdown files that expand into conversation prompts when invoked via `/command-name`
- Must include YAML frontmatter with `description`
- File name determines the slash command name (e.g., `example.md` → `/example`)
- Content after frontmatter is injected into the conversation context

**Agents** (`agents/`)
- Markdown files defining specialized agent behaviors for specific tasks
- Require both `name` and `description` in YAML frontmatter
- Structure should include: Purpose, Instructions, and Example Usage sections
- Used for autonomous task execution within Claude Code

**Skills** (`skills/`)
- Organized as subdirectories containing `SKILL.md` files
- Provide reusable capabilities invoked during conversations
- Each skill directory represents a discrete capability
- Frontmatter must include `name` and `description`

**Hooks** (`hooks/hooks.json`)
- JSON configuration for event-driven automation
- Available events: `user-prompt-submit`, `tool-use`, and others
- Each hook specifies a shell command and description
- Supports template variables (e.g., `{{tool_name}}`)

## Development Workflow

### Creating New Components

When adding functionality:
1. Determine the appropriate component type (command vs agent vs skill)
2. Create the file in the correct directory with proper frontmatter
3. For skills, create a new subdirectory under `skills/` first
4. Test the component by installing the plugin locally

### Local Testing

To test this plugin:
1. Create `.claude-plugin/marketplace.json` referencing this plugin's directory
2. Use Claude Code's plugin installation to load from the local marketplace
3. Verify components are accessible (commands via `/`, agents via Task tool, skills via invocation)

### Plugin Metadata Updates

When modifying `.claude-plugin/plugin.json`:
- Version changes require semantic versioning (MAJOR.MINOR.PATCH)
- Description changes affect how the plugin appears in listings
- Name changes require reinstallation for users

## RagnaRokkrr Integration Context

This plugin specifically targets RagnaRokkrr integration. Components should be designed to work with RagnaRokkrr's architecture and workflows. Replace example templates with concrete implementations aligned to RagnaRokkrr's needs whenever possible.
