# AGENTS.md

This file tells any autonomous coding agent how to work inside the `ragna-claude-marketplace` repository. It mirrors the high-level guidance in `CLAUDE.md` but states the rules in a concise, action-focused format.

## Mission
- Maintain a **local Claude Code marketplace** that distributes internal plugins for the RagnaRokkrr team.
- Keep `.claude-plugin/marketplace.json` accurate so Claude can add the marketplace via `/plugin marketplace add <path>`.

## Repo Map
```
.
├── .claude-plugin/marketplace.json   # Marketplace manifest (edit here first)
├── ragna-claude-plugins/             # Primary plugin (see its own CLAUDE/AGENTS files)
├── .spec/architecture/               # Example docs + ADRs for validation
├── CLAUDE.md                         # Human-facing guidelines
└── README.md                         # Marketplace overview
```

## Operating Rules
1. **Marketplace entries**: ensure each plugin object contains `name`, `source` (path relative to `.claude-plugin/marketplace.json`), and `description`. Update `version` fields if listed.
2. **Adding plugins**:
   - scaffold the plugin directory under repo root,
   - add the entry to `.claude-plugin/marketplace.json`,
   - document any install/test notes in the plugin’s README.
3. **Testing workflow**:
   ```bash
   /plugin marketplace add /home/npadilha/prj/ragna-claude-marketplace
   /plugin install <plugin-name>
   ```
4. **Architecture samples** under `.spec/architecture/` are reference material. Keep ADR IDs four digits and sequential (e.g., `adr-0006.md`).
5. **Git hygiene**: follow branch strategy documented in `CLAUDE.md` (`master`, `feature/*`, `agent/*`). Prefix commits with component tags when practical, e.g., `[marketplace] Update manifest`.

## Quick Checklist
- [ ] Validate JSON after editing manifests (`jq . file.json`).
- [ ] Keep CLAUDE/AGENTS pairs in sync (root + plugin).
- [ ] When scripting paths, remember marketplace commands execute relative to the manifest directory.
- [ ] Document new automation in the relevant README or CLAUDE file so future agents understand the change.
