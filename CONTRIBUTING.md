# Contributing to Claude for Skyflow

This guide is for developers who want to contribute to or extend this plugin.

## Repository Structure

This repository is organized as a Claude Code plugin marketplace:

```
/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace configuration
└── skyflow-plugin/           # Main Skyflow plugin
    ├── .claude-plugin/
    │   └── plugin.json       # Plugin metadata
    ├── .mcp.json             # MCP server configuration
    ├── commands/             # Custom slash commands
    │   └── hello.md
    ├── skills/               # Agent skills
    │   └── chat-message-anonymization-v0/
    │       └── skill.md
    ├── agents/               # Custom agents (optional)
    └── hooks/                # Event handlers (optional)
```

## Marketplace Configuration

The `.claude-plugin/marketplace.json` file at the root defines the marketplace and lists available plugins:

- `name`: The marketplace identifier
- `owner`: Marketplace owner information
- `plugins`: Array of plugin definitions with name, source path, and description

## Plugin Structure

Each plugin (e.g., `skyflow-plugin/`) contains:

- `.claude-plugin/plugin.json`: Plugin metadata (name, version, author, description)
- `.mcp.json`: MCP server configuration with endpoints and authentication
- `commands/`: Markdown files defining custom slash commands
- `skills/`: Agent skills with `skill.md` files
- `agents/`: Custom agent definitions (optional)
- `hooks/`: Event handlers via `hooks.json` (optional)

## MCP Server Configuration

The `.mcp.json` file in the plugin directory configures the MCP servers:

```json
{
  "skyflow-developer-mcp": {
    "type": "http",
    "url": "https://skyflow-mcp.dev/mcp?accountId=${SKYFLOW_ACCOUNT_ID}",
    "headers": {
      "Authorization": "Bearer ${SKYFLOW_BEARER_TOKEN}"
    }
  },
  "skyflow-runtime-mcp": {
    "type": "http",
    "url": "https://www.pii-mcp.dev/mcp?vaultId=${SKYFLOW_VAULT_ID}&vaultUrl=${SKYFLOW_VAULT_URL}",
    "headers": {
      "Authorization": "Bearer ${SKYFLOW_BEARER_TOKEN}"
    }
  }
}
```

## Adding New Commands

1. Create a new `.md` file in `skyflow-plugin/commands/`
2. The filename becomes the command name (e.g., `hello.md` → `/hello`)
3. Follow the existing command format

## Adding New Skills

1. Create a new directory in `skyflow-plugin/skills/`
2. Add a `skill.md` file with the skill definition
3. Follow the format in existing skills like `chat-message-anonymization-v0/`

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).
