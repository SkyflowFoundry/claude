# Claude Code Plugins for Skyflow

> [!WARNING]
> This is an experimental project in development. This project is not supported and offered under an MIT license.

A Claude Code plugin marketplace that publishes Skyflow's **MCP server** plugins. The two MCP servers are independent, optional add-ons, so you can install only what you need.

> **Looking for the Skyflow skills?** The `skyflow-skills` plugin moved to its own skills-only marketplace, [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills), so it can be reviewed and authorized independently of the MCP servers. See [Skyflow skills](#skyflow-skills) below.

- [Claude Code Plugins for Skyflow](#claude-code-plugins-for-skyflow)
  - [Plugins](#plugins)
  - [Quick Start](#quick-start)
  - [Skyflow skills](#skyflow-skills)
  - [Environment Variables Reference](#environment-variables-reference)
  - [Upgrading](#upgrading)
  - [Learn More](#learn-more)
  - [Contributing](#contributing)

## Plugins

| Plugin | What it does | Environment variables | Docs |
| ------ | ------------ | --------------------- | ---- |
| `skyflow-developer-mcp` | Developer MCP server — access to Skyflow documentation, skills, and integration resources | `SKYFLOW_BEARER_TOKEN`, `SKYFLOW_ACCOUNT_ID` | [README](skyflow-developer-mcp-plugin/README.md) |
| `skyflow-runtime-mcp` | Runtime MCP server (optional) — on-demand de-identification of PII in text via the Detect APIs | `SKYFLOW_BEARER_TOKEN`, `SKYFLOW_ACCOUNT_ID`, `SKYFLOW_VAULT_ID`, `SKYFLOW_VAULT_URL` | [README](skyflow-runtime-mcp-plugin/README.md) |

Most users want `skyflow-developer-mcp`. Add `skyflow-runtime-mcp` only if you need on-demand de-identification.

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

   Install the Developer MCP plugin:

   ```sh
   /plugin install skyflow-developer-mcp@skyflow-marketplace
   ```

   Optionally, install the Runtime MCP plugin:

   ```sh
   /plugin install skyflow-runtime-mcp@skyflow-marketplace
   ```

4. Set up environment variables for the MCP plugins, then restart Claude Code. The MCP plugins read the `SKYFLOW_*` variables described in their READMEs:

   - [skyflow-developer-mcp setup](skyflow-developer-mcp-plugin/README.md#set-up-environment-variables)
   - [skyflow-runtime-mcp setup](skyflow-runtime-mcp-plugin/README.md#set-up-environment-variables)

## Skyflow skills

The Skyflow skills (getting started, vault creation, REST API guidance, SDK migration, implementation planning, SDK quickstarts) are published from a separate, skills-only marketplace. It ships no MCP servers and needs no credentials, so it can be reviewed and allowlisted on its own:

```sh
/plugin marketplace add SkyflowFoundry/skyflow-skills
/plugin install skyflow-skills@skyflow-skills-marketplace
```

See [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills) for the skills, standalone skill downloads, and administrator allowlisting guidance.

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

Earlier versions shipped a single `skyflow` plugin that bundled both the skills and the MCP servers. The skills and MCP servers are now separate plugins:

- The **skills** moved to their own marketplace, [`SkyflowFoundry/skyflow-skills`](https://github.com/SkyflowFoundry/skyflow-skills) (plugin `skyflow-skills`).
- The **MCP servers** stay in this marketplace as `skyflow-developer-mcp` and `skyflow-runtime-mcp`.

If you previously installed `skyflow-skills` from this marketplace, Claude Code will notify you that it was removed here. Reinstall it from the skills marketplace, and (re)install the MCP plugins from this one:

```sh
/plugin marketplace update skyflow-marketplace
/plugin marketplace add SkyflowFoundry/skyflow-skills
/plugin install skyflow-skills@skyflow-skills-marketplace
/plugin install skyflow-developer-mcp@skyflow-marketplace
/plugin install skyflow-runtime-mcp@skyflow-marketplace   # optional
```

Your existing environment variables are unchanged; the MCP plugins read the same `SKYFLOW_*` variables you already configured.

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).

## Contributing

For developers who want to contribute to these plugins, see [CONTRIBUTING.md](CONTRIBUTING.md).
