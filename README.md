# Claude Code Plugins for Skyflow

> [!WARNING]
> This is an experimental project in development. This project is not supported and offered under an MIT license.

A Claude Code plugin marketplace that enables Skyflow's data privacy and protection capabilities. The marketplace publishes three plugins so you can install only what you need — the skills work standalone, and the two MCP servers are independent, optional add-ons.

- [Claude Code Plugins for Skyflow](#claude-code-plugins-for-skyflow)
  - [Plugins](#plugins)
  - [Quick Start](#quick-start)
  - [Standalone Skill Downloads](#standalone-skill-downloads)
  - [Environment Variables Reference](#environment-variables-reference)
  - [Upgrading](#upgrading)
  - [Learn More](#learn-more)
  - [Contributing](#contributing)

## Plugins

| Plugin | What it does | Environment variables | Docs |
| ------ | ------------ | --------------------- | ---- |
| `skyflow-skills` | Guided Skyflow workflows (vault creation, REST API guidance, SDK migration, implementation planning, SDK quickstarts) | None | [README](skyflow-skills-plugin/README.md) |
| `skyflow-developer-mcp` | Developer MCP server — access to Skyflow documentation, skills, and integration resources | `SKYFLOW_BEARER_TOKEN`, `SKYFLOW_ACCOUNT_ID` | [README](skyflow-developer-mcp-plugin/README.md) |
| `skyflow-runtime-mcp` | Runtime MCP server (optional) — on-demand de-identification of PII in text via the Detect APIs | `SKYFLOW_BEARER_TOKEN`, `SKYFLOW_ACCOUNT_ID`, `SKYFLOW_VAULT_ID`, `SKYFLOW_VAULT_URL` | [README](skyflow-runtime-mcp-plugin/README.md) |

Most users want `skyflow-skills` plus `skyflow-developer-mcp`. Add `skyflow-runtime-mcp` only if you need on-demand de-identification.

## Quick Start

> **Note:** You must have Claude Code installed first. If you haven't installed it yet, see the [Claude Code installation guide](https://code.claude.com/docs/en/overview).

1. Open Claude Code:

   ```sh
   claude
   ```

2. Add the marketplace:

   ```sh
   /plugin marketplace add SkyflowFoundry/claude
   ```

3. Install the plugins you want.

   Install the skills plugin:

   ```sh
   /plugin install skyflow-skills@skyflow-marketplace
   ```

   Install the Developer MCP plugin:

   ```sh
   /plugin install skyflow-developer-mcp@skyflow-marketplace
   ```

   Optionally, install the Runtime MCP plugin:

   ```sh
   /plugin install skyflow-runtime-mcp@skyflow-marketplace
   ```

4. Set up environment variables for the MCP plugins, then restart Claude Code. The `skyflow-skills` plugin needs none; the MCP plugins read the `SKYFLOW_*` variables described in their READMEs:

   - [skyflow-developer-mcp setup](skyflow-developer-mcp-plugin/README.md#set-up-environment-variables)
   - [skyflow-runtime-mcp setup](skyflow-runtime-mcp-plugin/README.md#set-up-environment-variables)

## Standalone Skill Downloads

The `skyflow-skills` plugin is the easiest way to get the skills in Claude Code. If you instead want a single skill as a portable file — to drop into another project, share, or use with a different Agent Skills-compatible harness — each skill is also published as a standalone `.zip` on the [Releases page](https://github.com/SkyflowFoundry/claude/releases/latest).

Each archive unzips to a self-contained skill folder (`<skill-name>/SKILL.md` plus its resources). To install one manually:

```sh
# Download the latest build of a skill (stable URL always points at the newest release)
curl -L -O https://github.com/SkyflowFoundry/claude/releases/latest/download/create-vault.zip

# Unzip into your user skills directory (or a project's .claude/skills/)
unzip create-vault.zip -d ~/.claude/skills/
```

Available skills: `call-rest-apis`, `create-vault`, `migrate-sdk-v1-to-v2`, `plan-skyflow-implementation`, `quickstart-js-browser`, `quickstart-node`. A `SHA256SUMS.txt` is attached to each release so you can verify downloads.

> These zips are build artifacts generated from the same skills in this repo — the plugin and the standalone downloads are always in sync.

## Environment Variables Reference

These variables are read by the MCP plugins. See each plugin's README for step-by-step setup.

| Variable               | Description                                                                       | Required by |
| ---------------------- | --------------------------------------------------------------------------------- | ----------- |
| `SKYFLOW_BEARER_TOKEN` | A Skyflow API key (recommended), personal access token, or generated bearer token | `skyflow-developer-mcp`, `skyflow-runtime-mcp` |
| `SKYFLOW_ACCOUNT_ID`   | Your Skyflow account identifier                                                   | `skyflow-developer-mcp`, `skyflow-runtime-mcp` |
| `SKYFLOW_VAULT_ID`     | Your Detect vault identifier                                                      | `skyflow-runtime-mcp` |
| `SKYFLOW_VAULT_URL`    | Your vault URL endpoint                                                           | `skyflow-runtime-mcp` |

**Note:** The `skyflow-runtime-mcp` server requires that your bearer token has 'Vault Owner' access to a Detect vault (a vault created with the detect template).

## Upgrading

Earlier versions shipped a single `skyflow` plugin that bundled both the skills and the MCP servers. That plugin has been **renamed to `skyflow-skills`** and now contains skills only; the MCP servers moved to the separate `skyflow-developer-mcp` and `skyflow-runtime-mcp` plugins. If you installed the old `skyflow` plugin, uninstall it and install the new plugins:

```sh
/plugin marketplace update skyflow-marketplace
/plugin uninstall skyflow@skyflow-marketplace
/plugin install skyflow-skills@skyflow-marketplace
/plugin install skyflow-developer-mcp@skyflow-marketplace
/plugin install skyflow-runtime-mcp@skyflow-marketplace   # optional
```

Your existing environment variables are unchanged; the MCP plugins read the same `SKYFLOW_*` variables you already configured.

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).

## Contributing

For developers who want to contribute to these plugins, see [CONTRIBUTING.md](CONTRIBUTING.md).
