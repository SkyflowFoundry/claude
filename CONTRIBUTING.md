# Contributing to Claude for Skyflow

This guide is for developers who want to contribute to or extend this plugin.

## Repository Structure

This repository is organized as a Claude Code plugin marketplace:

```
/
├── README.md                             # Marketplace overview
├── .claude-plugin/
│   └── marketplace.json                  # Marketplace configuration (lists all plugins)
├── skyflow-skills-plugin/                # Skills plugin (no MCP servers)
│   ├── .claude-plugin/
│   │   └── plugin.json                   # Plugin metadata
│   ├── README.md                         # Plugin docs
│   └── skills/                           # Agent skills
│       ├── call-rest-apis/
│       │   └── SKILL.md
│       ├── create-vault/
│       │   └── SKILL.md
│       ├── migrate-sdk-v1-to-v2/
│       │   └── SKILL.md
│       └── plan-skyflow-implementation/
│           └── SKILL.md
├── skyflow-developer-mcp-plugin/         # Developer MCP server plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── README.md                         # Plugin docs
│   └── .mcp.json                         # Developer MCP server config
└── skyflow-runtime-mcp-plugin/           # Runtime MCP server plugin (optional)
    ├── .claude-plugin/
    │   └── plugin.json
    ├── README.md                         # Plugin docs
    └── .mcp.json                         # Runtime MCP server config
```

The skills and the MCP servers are split into separate plugins so each can be installed and versioned independently.

## Marketplace Configuration

The `.claude-plugin/marketplace.json` file at the root defines the marketplace and lists available plugins:

- `name`: The marketplace identifier
- `owner`: Marketplace owner information
- `plugins`: Array of plugin definitions with name, source path, and description. The marketplace currently lists three plugins: `skyflow-skills`, `skyflow-developer-mcp`, and `skyflow-runtime-mcp`.

A plugin's `name` must match the `name` in its own `.claude-plugin/plugin.json`, and `source` is a path relative to the repository root (e.g. `./skyflow-developer-mcp-plugin`).

## Plugin Structure

This marketplace uses two kinds of plugin:

**Skills plugin (`skyflow-skills-plugin/`):**

- `.claude-plugin/plugin.json`: Plugin metadata (name, version, author, description)
- `skills/`: Agent skills, each in its own directory with a `SKILL.md` file

**MCP server plugins (`skyflow-developer-mcp-plugin/`, `skyflow-runtime-mcp-plugin/`):**

- `.claude-plugin/plugin.json`: Plugin metadata
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

## Adding New Skills

Skills are guided workflows that help Claude assist users with Skyflow tasks. They provide structured documentation, examples, and references that Claude can use when helping users implement features.

### Skill Directory Structure

Create a new directory in `skyflow-skills-plugin/skills/` with the following structure:

```
skyflow-skills-plugin/skills/your-skill-name/
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
