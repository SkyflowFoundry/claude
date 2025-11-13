# Claude for Skyflow

A collection of skills, commands, subagents, prompts, etc for use with Claude Code when developing with Skyflow.

## Structure

This repository is organized as a Claude Code plugin marketplace:

```
/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace configuration
├── skyflow-plugin/            # Main Skyflow plugin
│   ├── .claude-plugin/
│   │   └── plugin.json       # Plugin metadata
│   ├── commands/              # Custom slash commands
│   ├── agents/                # Custom agents (optional)
│   ├── skills/                # Agent skills (optional)
│   └── hooks/                 # Event handlers (optional)
└── skills/                    # Legacy skills directory
```

### Marketplace Configuration

The `.claude-plugin/marketplace.json` file at the root defines the marketplace and lists available plugins:
- `name`: The marketplace identifier
- `owner`: Marketplace owner information
- `plugins`: Array of plugin definitions with name, source path, and description

### Plugin Structure

Each plugin (e.g., `skyflow-plugin/`) contains:
- `.claude-plugin/plugin.json`: Plugin metadata (name, version, author, description)
- `commands/`: Markdown files defining custom slash commands
- `agents/`: Custom agent definitions (optional)
- `skills/`: Agent skills with `SKILL.md` files (optional)
- `hooks/`: Event handlers via `hooks.json` (optional)
