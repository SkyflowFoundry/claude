# Claude Code Plugin for Skyflow

> [!WARNING]
> This is an experimental project in development. This project is not supported and offered under an MIT license.

A plugin for Claude Code that enables Skyflow's data privacy and protection capabilities.

- [Claude Code Plugin for Skyflow](#claude-code-plugin-for-skyflow)
  - [Quick Start](#quick-start)
    - [Step 1: Install the Plugins](#step-1-install-the-plugins)
    - [Step 2: Set Up Environment Variables](#step-2-set-up-environment-variables)
  - [Environment Variables Reference](#environment-variables-reference)
  - [About the MCP Servers](#about-the-mcp-servers)
  - [Skills](#skills)
    - [Plan Skyflow Implementation](#plan-skyflow-implementation)
    - [Create Vault](#create-vault)
    - [Call REST APIs](#call-rest-apis)
    - [Migrate SDK V1 to V2](#migrate-sdk-v1-to-v2)
  - [Upgrading](#upgrading)
  - [Troubleshooting](#troubleshooting)
  - [Learn More](#learn-more)
  - [Contributing](#contributing)

## Quick Start

### Step 1: Install the Plugins

> **Note:** You must have Claude Code installed first. If you haven't installed it yet, see the [Claude Code installation guide](https://code.claude.com/docs/en/overview).

The marketplace publishes three plugins, so you can install only what you need:

- **`skyflow`** — Skills (guided workflows). Useful on its own and has no environment variable requirements.
- **`skyflow-developer-mcp`** — The Developer MCP server. Most users want this for live access to Skyflow documentation and resources.
- **`skyflow-runtime-mcp`** — The Runtime MCP server (optional), for on-demand de-identification of PII in text.

1. Open Claude Code:

   ```sh
   claude
   ```

2. Add the marketplace:

   ```sh
   /plugin marketplace add SkyflowFoundry/claude
   ```

3. Install the plugins you want:

   ```sh
   /plugin install skyflow@skyflow-marketplace
   /plugin install skyflow-developer-mcp@skyflow-marketplace
   /plugin install skyflow-runtime-mcp@skyflow-marketplace   # optional
   ```

4. Exit Claude Code:

   ```sh
   /exit
   ```

### Step 2: Set Up Environment Variables

> **Note:** These instructions are for macOS. Windows and Linux users may need to adjust the shell configuration file and commands accordingly.

The MCP servers ship as two separate plugins so you only configure what you install. The `skyflow` skills plugin needs no environment variables.

**Developer MCP Server** (`skyflow-developer-mcp` plugin) - Provides access to Skyflow developer documentation, skills, and helpful resources for people integrating or implementing Skyflow. This is all most users need to get started, and it only requires `SKYFLOW_BEARER_TOKEN` and `SKYFLOW_ACCOUNT_ID`.

**Runtime MCP Server** (`skyflow-runtime-mcp` plugin, optional) - Wraps the Skyflow Detect APIs for removing, redacting, tokenizing, and de-identifying (as well as re-identifying) PII in unstructured text on demand. Install this plugin only if you need that capability; it additionally requires `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL`. If you install it without setting those, the server will fail to initialize and Claude will display an error on startup.

#### Add the environment variables

Run these commands in Terminal, replacing the placeholder values with your actual Skyflow credentials:

```bash
# Required for Developer MCP
echo 'export SKYFLOW_BEARER_TOKEN="your-token-here"' >> ~/.zshrc
echo 'export SKYFLOW_ACCOUNT_ID="your-account-id-here"' >> ~/.zshrc
```

To obtain your bearer token and account ID, see the [Skyflow API Authentication documentation](https://docs.skyflow.com/docs/fundamentals/api-authentication).

**Note:** The bearer token can be a Skyflow API key (recommended for long-lived access), personal access token, or generated bearer token.

**Optional:** If you want to use the Runtime MCP server for de-identification, also run:

```bash
# Optional: Only needed for Runtime MCP server
echo 'export SKYFLOW_VAULT_ID="your-vault-id-here"' >> ~/.zshrc
echo 'export SKYFLOW_VAULT_URL="your-vault-url-here"' >> ~/.zshrc
```

#### Apply the changes

**Quit Terminal completely** (Cmd + Q), then reopen it.

#### Verify your setup

Run this command to confirm your environment variables are set:

```bash
echo $SKYFLOW_BEARER_TOKEN
```

If it prints your token, you're ready to restart Claude Code. If it prints nothing, revisit the steps above.

## Environment Variables Reference

| Variable               | Description                                                                       | Required |
| ---------------------- | --------------------------------------------------------------------------------- | -------- |
| `SKYFLOW_BEARER_TOKEN` | A Skyflow API key (recommended), personal access token, or generated bearer token | Yes      |
| `SKYFLOW_ACCOUNT_ID`   | Your Skyflow account identifier                                                   | Yes      |
| `SKYFLOW_VAULT_ID`     | Your detect vault identifier                                                      | No       |
| `SKYFLOW_VAULT_URL`    | Your vault URL endpoint                                                           | No       |

**Note:** `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL` are only needed if you install the `skyflow-runtime-mcp` plugin. The Runtime MCP server also requires that your bearer token has 'Vault Owner' access to a Detect vault (a vault created with the detect template).

## About the MCP Servers

Each MCP server is published as its own plugin, so you can install them independently of the `skyflow` skills.

- **Developer MCP** (`skyflow-developer-mcp` plugin, `https://skyflow-mcp.dev`) - Access to Skyflow developer documentation, skills, and resources for integrating and implementing Skyflow.

- **Runtime MCP** (`skyflow-runtime-mcp` plugin, `https://www.pii-mcp.dev`) - Wraps the Skyflow Detect APIs for de-identifying and re-identifying PII in unstructured text. Use this when you need to remove, redact, or tokenize sensitive data on demand.

  > **Note:** Because the Runtime MCP is a separate, optional plugin, developer-only users never see its startup error—simply don't install `skyflow-runtime-mcp`. If you do install it without configuring `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL`, you may see an error from the server on startup. To enable it, create a Detect vault in Skyflow and configure these environment variables.

## Skills

Skills are guided workflows that help Claude assist you with common Skyflow tasks. They provide structured documentation, sample schemas, and API examples that Claude can reference when helping you implement Skyflow features.

### Plan Skyflow Implementation

The **plan-skyflow-implementation** skill guides you through planning a complete Skyflow implementation using the Define-Build-Go Live framework. It helps you assess requirements (data inventory, schema design, environment setup), plan technical integration (authentication, SDK integration, access controls, testing), and prepare for production (security review, data migration, launch). The skill includes use case classification, phase-specific checklists, tokenization decision trees, implementation templates, and security review guidance.

### Create Vault

The **create-vault** skill guides you through creating Skyflow vaults programmatically using the Management API. It covers three approaches: using pre-built templates (Quickstart, Payment, PIIData, CustomerIdentity, Plaid), uploading a custom schema, or starting from scratch. The skill includes complete API examples for listing templates, creating vaults, and updating schemas, along with comprehensive documentation on configuring field tags for tokenization policies, redaction/DLP settings, validation rules, and compliance classifications (GDPR, CCPA, HIPAA, etc.). Sample vault schemas are provided for common use cases like payment processing, customer identity management, and PII storage.

### Call REST APIs

The **call-rest-apis** skill provides expertise on Skyflow REST APIs including management APIs, data APIs, and detect APIs. It covers API endpoints, request/response formats, authentication methods, and code examples. The skill includes an API quick reference table, OpenAPI specifications for data, detect, and management APIs, authentication guidance for bearer tokens and service accounts, error handling patterns, rate limiting information, and links to SDK documentation.

### Migrate SDK V1 to V2

The **migrate-sdk-v1-to-v2** skill guides you through migrating from Skyflow V1 SDKs to V2 SDKs. It covers authentication changes, client initialization updates, and request/response structure changes with SDK-specific migration patterns for Node.js, Python, Java, and Go. The skill includes V1 identification patterns, breaking changes documentation, a migration workflow, before/after code examples for common patterns, a troubleshooting guide, and test strategies.

## Upgrading

Earlier versions bundled the MCP servers inside the `skyflow` plugin. They now ship as the separate `skyflow-developer-mcp` and `skyflow-runtime-mcp` plugins. If you installed `skyflow` before this change, updating it **removes** the MCP servers from your environment—you must install the MCP plugins separately to get those tools back:

```sh
/plugin marketplace update skyflow-marketplace
/plugin install skyflow-developer-mcp@skyflow-marketplace
/plugin install skyflow-runtime-mcp@skyflow-marketplace   # optional
```

Your existing environment variables are unchanged; the MCP plugins read the same `SKYFLOW_*` variables you already configured.

## Troubleshooting

**"Environment variable not set" errors:**

- Make sure you ran the `echo` commands to add the variables to `~/.zshrc`.
- Make sure you quit and reopened Terminal (not just opened a new tab).
- Make sure you restarted Claude Code.

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).

## Contributing

For developers who want to contribute to this plugin, see [CONTRIBUTING.md](CONTRIBUTING.md).
