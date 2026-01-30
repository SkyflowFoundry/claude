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

Skills are guided workflows that help Claude assist users with Skyflow tasks. They provide structured documentation, examples, and references that Claude can use when helping users implement features.

### Skill Directory Structure

Create a new directory in `skyflow-plugin/skills/` with the following structure:

```
skyflow-plugin/skills/your-skill-name/
├── SKILL.md                    # Main skill file (required)
├── supporting-doc.md           # Additional documentation (optional)
├── samples/                    # Sample files (optional)
│   ├── example-1.json
│   └── example-2.json
└── schemas/                    # Validation schemas (optional)
    └── schema.json
```

### SKILL.md Format

The main skill file must include a YAML frontmatter header followed by the skill content:

```markdown
---
name: your-skill-name
description: Brief description of what the skill helps users accomplish.
---

# Skill Title

Overview paragraph explaining the purpose.

## Prerequisites
## Step 1: First Step
## Step 2: Second Step
...
## Troubleshooting
## Related Documentation
```

### Best Practices

#### Structure and Organization

- Keep the skill name lowercase with hyphens (e.g., `create-vault`, `detect-pii`)
- Start with an Overview section explaining what the skill accomplishes
- Include a Prerequisites section with required accounts, tokens, and tools
- Use numbered steps for the main workflow
- End with Troubleshooting and Related Documentation sections

#### API-First Approach

- Prefer API examples over Studio UI instructions where possible
- Include complete, copy-pastable curl commands with environment variables
- Document required environment variables at the start
- Note explicitly when Studio UI is required for certain operations

#### Documentation Quality

- Use tables for reference information (data types, tag values, templates)
- Include realistic examples showing complete configurations
- Link to supporting documentation files for detailed references
- Keep the main SKILL.md scannable; put comprehensive details in supporting docs

#### Supporting Files

- Place sample schemas/configs in a `samples/` or descriptive subdirectory
- Include validation schemas (JSONSchema) when applicable
- Use relative links to reference supporting files from SKILL.md
- Provide samples for common use cases (e.g., quickstart, payment, PII)

#### Example Field Configuration

When documenting configurable options, show a complete example:

```json
{
  "name": "email",
  "datatype": "DT_STRING",
  "tags": [
    { "name": "skyflow.options.default_token_policy", "values": ["DETERMINISTIC_UUID"] },
    { "name": "skyflow.options.default_dlp_policy", "values": ["MASK"] },
    { "name": "skyflow.validation.regular_exp", "values": ["^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"] }
  ]
}
```

### Updating the README

After creating a new skill, add it to the Skills section in the main [README.md](README.md):

1. Add a subsection under `## Skills` with the skill name as a heading
2. Write a paragraph describing what the skill does and its key features
3. The table of contents will be updated automatically if using a markdown formatter

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).
