# Claude Code Plugin for Skyflow

> [!WARNING]
> This is an experimental project in development. This project is not supported and offered under an MIT license.

A plugin for Claude Code that enables Skyflow's data privacy and protection capabilities.

- [Claude Code Plugin for Skyflow](#claude-code-plugin-for-skyflow)
  - [Quick Start](#quick-start)
    - [Step 1: Install the Plugin](#step-1-install-the-plugin)
    - [Step 2: Set Up Environment Variables](#step-2-set-up-environment-variables)
      - [Open your shell configuration file](#open-your-shell-configuration-file)
      - [Add the environment variables](#add-the-environment-variables)
      - [Save and exit](#save-and-exit)
      - [Apply the changes](#apply-the-changes)
  - [Environment Variables Reference](#environment-variables-reference)
    - [Required for Developer MCP](#required-for-developer-mcp)
    - [Optional: only required for Runtime MCP](#optional-only-required-for-runtime-mcp)
  - [About the MCP Servers](#about-the-mcp-servers)
  - [Skills](#skills)
    - [Create Vault](#create-vault)
  - [Troubleshooting](#troubleshooting)
  - [Learn More](#learn-more)
  - [Contributing](#contributing)

## Quick Start

### Step 1: Install the Plugin

> **Note:** You must have Claude Code installed first. If you haven't installed it yet, see the [Claude Code installation guide](https://code.claude.com/docs/en/overview).

1. Open Claude Code:

   ```sh
   claude
   ```

2. Install the marketplace and plugin:

   ```sh
   /plugin marketplace add SkyflowFoundry/claude
   /plugin install skyflow@skyflow-marketplace
   ```

3. Exit Claude Code:

   ```sh
   /exit
   ```

### Step 2: Set Up Environment Variables

The Skyflow plugin connects to two MCP servers. You can start with just the Developer MCP server—the Runtime MCP server is optional.

**Developer MCP Server** - Provides access to Skyflow developer documentation, skills, and helpful resources for people integrating or implementing Skyflow. This is all most users need to get started.

**Runtime MCP Server** (optional) - Wraps the Skyflow Detect APIs for removing, redacting, tokenizing, and de-identifying (as well as re-identifying) PII in unstructured text on demand. If you do not set `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL`, this server will fail to initialize and Claude will display an error on startup.

#### Open your shell configuration file

1. Open Terminal.
2. Run this command to edit your configuration file:

   ```bash
   nano ~/.zshrc
   ```

   (This opens the nano text editor. If the file doesn't exist, it will be created.)

#### Add the environment variables

1. Use the arrow keys to scroll to the bottom of the file.
2. Copy and paste these lines:

   ```bash
   # Skyflow Configuration (required for Developer MCP)
   export SKYFLOW_BEARER_TOKEN="your-token-here"
   export SKYFLOW_ACCOUNT_ID="your-account-id-here"

   # Optional: Only needed for Runtime MCP server
   # export SKYFLOW_VAULT_ID="your-vault-id-here"
   # export SKYFLOW_VAULT_URL="your-vault-url-here"
   ```

3. Replace each `"your-...-here"` value with your actual Skyflow credentials. To obtain your bearer token, account ID, vault ID, and vault URL, see the [Skyflow API Authentication documentation](https://docs.skyflow.com/docs/fundamentals/api-authentication).

**Note:** The bearer token can be a Skyflow API key (recommended for long-lived access), personal access token, or generated bearer token.

#### Save and exit

1. Press `Ctrl + O` (the letter O) to save the file.
2. Press `Enter` to confirm the filename.
3. Press `Ctrl + X` to exit nano.

#### Apply the changes

1. **Quit Terminal completely** (Cmd + Q), then reopen it.
2. **Restart Claude Code** to pick up the new environment variables.

## Environment Variables Reference

### Required for Developer MCP

| Variable               | Description                                                                       |
| ---------------------- | --------------------------------------------------------------------------------- |
| `SKYFLOW_BEARER_TOKEN` | A Skyflow API key (recommended), personal access token, or generated bearer token |
| `SKYFLOW_ACCOUNT_ID`   | Your Skyflow account identifier                                                   |

### Optional: only required for Runtime MCP

| Variable            | Description                  |
| ------------------- | ---------------------------- |
| `SKYFLOW_VAULT_ID`  | Your detect vault identifier |
| `SKYFLOW_VAULT_URL` | Your vault URL endpoint      |

**Note:** The Runtime MCP server requires that your bearer token has 'Vault Owner' access to a Detect vault (a vault created with the detect template).

## About the MCP Servers

- **Developer MCP** (`https://skyflow-mcp.dev`) - Access to Skyflow developer documentation, skills, and resources for integrating and implementing Skyflow.

- **Runtime MCP** (`https://www.pii-mcp.dev`) - Wraps the Skyflow Detect APIs for de-identifying and re-identifying PII in unstructured text. Use this when you need to remove, redact, or tokenize sensitive data on demand.

## Skills

Skills are guided workflows that help Claude assist you with common Skyflow tasks. They provide structured documentation, sample schemas, and API examples that Claude can reference when helping you implement Skyflow features.

### Create Vault

The **create-vault** skill guides you through creating Skyflow vaults programmatically using the Management API. It covers three approaches: using pre-built templates (Quickstart, Payment, PIIData, CustomerIdentity, Plaid), uploading a custom schema, or starting from scratch. The skill includes complete API examples for listing templates, creating vaults, and updating schemas, along with comprehensive documentation on configuring field tags for tokenization policies, redaction/DLP settings, validation rules, and compliance classifications (GDPR, CCPA, HIPAA, etc.). Sample vault schemas are provided for common use cases like payment processing, customer identity management, and PII storage.

## Troubleshooting

**"Environment variable not set" errors:**

- Make sure you saved the `~/.zshrc` file after adding the variables.
- Make sure you quit and reopened Terminal (not just opened a new tab).
- Make sure you restarted Claude Code.

**Verify your variables are set:**

Run this in Terminal to check:

```bash
echo $SKYFLOW_BEARER_TOKEN
```

If it prints your token, the variable is set correctly. If it prints nothing, the variable is not set.

**File doesn't exist error:**

If you get an error that `~/.zshrc` doesn't exist, create it first:

```bash
touch ~/.zshrc
```

Then run `nano ~/.zshrc` again.

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).

## Contributing

For developers who want to contribute to this plugin, see [CONTRIBUTING.md](CONTRIBUTING.md).
