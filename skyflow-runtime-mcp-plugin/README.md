# Skyflow Runtime MCP

The Runtime MCP server (`https://www.pii-mcp.dev`) for Claude Code. It wraps the Skyflow Detect APIs for removing, redacting, tokenizing, and de-identifying (as well as re-identifying) PII in unstructured text on demand. Use this when you need to remove, redact, or tokenize sensitive data on demand.

**This plugin is optional.** Install it only if you need on-demand de-identification. Because it is a separate plugin, developer-only users never see its startup error — simply don't install it.

Part of the [Skyflow marketplace](../README.md).

## Install

```sh
/plugin marketplace add SkyflowFoundry/claude
/plugin install skyflow-runtime-mcp@skyflow-marketplace
```

After installing, set the environment variables below and restart Claude Code.

> **Note:** If you install this plugin without configuring `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL`, the server will fail to initialize and Claude will display an error on startup. To enable it, create a Detect vault in Skyflow and configure these environment variables.

## Environment Variables

| Variable               | Description                                                                       | Required |
| ---------------------- | --------------------------------------------------------------------------------- | -------- |
| `SKYFLOW_BEARER_TOKEN` | A Skyflow API key (recommended), personal access token, or generated bearer token | Yes      |
| `SKYFLOW_ACCOUNT_ID`   | Your Skyflow account identifier                                                   | Yes      |
| `SKYFLOW_VAULT_ID`     | Your Detect vault identifier                                                      | Yes      |
| `SKYFLOW_VAULT_URL`    | Your vault URL endpoint                                                           | Yes      |

`SKYFLOW_BEARER_TOKEN` and `SKYFLOW_ACCOUNT_ID` are shared with the [`skyflow-developer-mcp`](../skyflow-developer-mcp-plugin/README.md) plugin; if you already configured those, you only need to add `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL`.

**Note:** The Runtime MCP server requires that your bearer token has 'Vault Owner' access to a Detect vault (a vault created with the detect template).

## Set Up Environment Variables

> **Note:** These instructions are for macOS. Windows and Linux users may need to adjust the shell configuration file and commands accordingly.

### Add the environment variables

Run these commands in Terminal, replacing the placeholder values with your actual Skyflow credentials:

```bash
echo 'export SKYFLOW_BEARER_TOKEN="your-token-here"' >> ~/.zshrc
echo 'export SKYFLOW_ACCOUNT_ID="your-account-id-here"' >> ~/.zshrc
echo 'export SKYFLOW_VAULT_ID="your-vault-id-here"' >> ~/.zshrc
echo 'export SKYFLOW_VAULT_URL="your-vault-url-here"' >> ~/.zshrc
```

To obtain your bearer token and account ID, see the [Skyflow API Authentication documentation](https://docs.skyflow.com/docs/fundamentals/api-authentication).

### Apply the changes

**Quit Terminal completely** (Cmd + Q), then reopen it.

### Verify your setup

Run this command to confirm your environment variables are set:

```bash
echo $SKYFLOW_VAULT_ID
```

If it prints your vault ID, you're ready to restart Claude Code. If it prints nothing, revisit the steps above.

## Troubleshooting

**"Environment variable not set" errors:**

- Make sure you ran the `echo` commands to add the variables to `~/.zshrc`.
- Make sure you quit and reopened Terminal (not just opened a new tab).
- Make sure you restarted Claude Code.

**Runtime MCP server error on startup:**

- Confirm `SKYFLOW_VAULT_ID` and `SKYFLOW_VAULT_URL` are set and point to a Detect vault.
- Confirm your bearer token has 'Vault Owner' access to that Detect vault.
