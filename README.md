# Claude for Skyflow

A collection of skills, commands, subagents, prompts, etc for use with Claude Code when developing with Skyflow.

## Using this Plugin

### Add the Marketplace

In Claude Code, add this marketplace:

```
/plugin marketplace add SkyflowFoundry/claude
```

### Browse and Install Plugins

Browse available plugins from this marketplace:

```
/plugin
```

Select and install the plugins you want to use.

### Configure Environment Variables

The Skyflow plugin includes an MCP server that requires two environment variables to be set:

**macOS/Linux:**

Add these to your shell profile (`~/.bashrc`, `~/.zshrc`, or `~/.bash_profile`):

```bash
export SKYFLOW_BEARER_TOKEN="your-token-here"
export SKYFLOW_ACCOUNT_ID="your-account-id-here"
```

Then reload your shell:

```bash
source ~/.zshrc  # or ~/.bashrc, ~/.bash_profile
```

**Windows (PowerShell):**

```powershell
$env:SKYFLOW_BEARER_TOKEN="your-token-here"
$env:SKYFLOW_ACCOUNT_ID="your-account-id-here"
```

To make these permanent, add them to your PowerShell profile or use System Environment Variables.

**Windows (Command Prompt):**

```cmd
set SKYFLOW_BEARER_TOKEN=your-token-here
set SKYFLOW_ACCOUNT_ID=your-account-id-here
```

### Learn More

For complete documentation on Claude Code plugins, see [https://code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)

## Structure

This repository is organized as a Claude Code plugin marketplace:

```
/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace configuration
└── skyflow-plugin/            # Main Skyflow plugin
    ├── .claude-plugin/
    │   └── plugin.json       # Plugin metadata
    ├── commands/              # Custom slash commands
    │   └── hello.md
    ├── skills/                # Agent skills
    │   └── chat-message-anonymization-v0/
    │       └── skill.md
    ├── agents/                # Custom agents (optional)
    └── hooks/                 # Event handlers (optional)
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
