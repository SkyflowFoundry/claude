# Contributing to Claude for Skyflow

This guide is for developers who want to contribute to or extend the Skyflow MCP plugins.

> **Contributing a skill?** The Skyflow skills live in a separate repository, [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills). Open skill changes there, not here.

## Repository Structure

This repository is a Claude Code plugin marketplace that publishes the Skyflow MCP server plugins:

```
/
├── README.md                             # Marketplace overview
├── .claude-plugin/
│   └── marketplace.json                  # Marketplace configuration (lists the MCP plugins)
├── skyflow-developer-mcp-plugin/         # Developer MCP server plugin
│   ├── .claude-plugin/
│   │   └── plugin.json                   # Plugin metadata
│   ├── README.md                         # Plugin docs
│   └── .mcp.json                         # Developer MCP server config
└── skyflow-runtime-mcp-plugin/           # Runtime MCP server plugin (optional)
    ├── .claude-plugin/
    │   └── plugin.json
    ├── README.md                         # Plugin docs
    └── .mcp.json                         # Runtime MCP server config
```

The skills and the MCP servers are maintained as separate marketplaces so each can be reviewed, authorized, and versioned independently. The skills-only marketplace is [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills).

## Marketplace Configuration

The `.claude-plugin/marketplace.json` file at the root defines the marketplace and lists available plugins:

- `name`: The marketplace identifier (`skyflow-marketplace`)
- `owner`: Marketplace owner information
- `plugins`: Array of plugin definitions with name, source path, and description. This marketplace lists two plugins: `skyflow-developer-mcp` and `skyflow-runtime-mcp`.
- `renames`: Migration map for plugins that were renamed or removed. `skyflow-skills` is mapped to `null` because it moved to the [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills) marketplace; existing users get an automatic "removed from this marketplace" notice and can reinstall it from the skills marketplace.

A plugin's `name` must match the `name` in its own `.claude-plugin/plugin.json`, and `source` is a path relative to the repository root (e.g. `./skyflow-developer-mcp-plugin`).

## Plugin Structure

Both plugins in this marketplace are MCP server plugins:

- `.claude-plugin/plugin.json`: Plugin metadata (name, version, author, description)
- `.mcp.json`: A single MCP server's configuration (endpoint and authentication)

Each plugin also carries its own `README.md` at its root documenting installation and configuration; the root [README.md](README.md) is a marketplace overview that links to them.

A plugin only requires `.claude-plugin/plugin.json`; a root-level `.mcp.json` is auto-discovered, so MCP-only plugins need no extra wiring. (`commands/`, `agents/`, and `hooks/` are also supported by Claude Code but are not currently used in this repo.)

## MCP Server Configuration

Each MCP plugin contains a `.mcp.json` at its root defining one server. For example, `skyflow-developer-mcp-plugin/.mcp.json`:

```json
{
  "skyflow-developer-mcp": {
    "type": "http",
    "url": "https://skyflow-mcp.dev/mcp?accountId=${SKYFLOW_ACCOUNT_ID}",
    "headers": {
      "Authorization": "Bearer ${SKYFLOW_BEARER_TOKEN}"
    }
  }
}
```

and `skyflow-runtime-mcp-plugin/.mcp.json`:

```json
{
  "skyflow-runtime-mcp": {
    "type": "http",
    "url": "https://www.pii-mcp.dev/mcp?vaultId=${SKYFLOW_VAULT_ID}&vaultUrl=${SKYFLOW_VAULT_URL}",
    "headers": {
      "Authorization": "Bearer ${SKYFLOW_BEARER_TOKEN}"
    }
  }
}
```

The `${...}` placeholders are substituted from the user's shell environment when the server starts.

## Adding a new MCP server plugin

1. Create a new `<name>-plugin/` directory at the repository root.
2. Add `.claude-plugin/plugin.json` with the plugin metadata (`name`, `description`, `version`, `author`).
3. Add a root-level `.mcp.json` defining the single server (see above). Reference secrets through `${...}` environment placeholders — never commit tokens.
4. Add a `README.md` documenting installation and environment variables.
5. Add an entry to `.claude-plugin/marketplace.json` with the plugin `name`, `source` (e.g. `./<name>-plugin`), and `description`.
6. Validate with `claude plugin validate .` (or `/plugin validate .` inside Claude Code).

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).
