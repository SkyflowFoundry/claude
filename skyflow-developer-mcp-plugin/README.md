# Skyflow Developer MCP

The Developer MCP server (`https://skyflow-mcp.dev`) for Claude Code. It provides access to Skyflow developer documentation, skills, and helpful resources for people integrating or implementing Skyflow. This is all most users need to get started.

Part of the [Skyflow marketplace](../README.md).

## Install

Add the Skyflow marketplace:

```sh
/plugin marketplace add SkyflowFoundry/claude
```

Install the plugin:

```sh
/plugin install skyflow-developer-mcp@skyflow-marketplace
```

After installing, set the environment variables below and restart Claude Code.

## Environment Variables

| Variable               | Description                                                                       | Required |
| ---------------------- | --------------------------------------------------------------------------------- | -------- |
| `SKYFLOW_BEARER_TOKEN` | A Skyflow API key (recommended), personal access token, or generated bearer token | Yes      |
| `SKYFLOW_ACCOUNT_ID`   | Your Skyflow account identifier                                                   | Yes      |

To obtain your bearer token and account ID, see the [Skyflow API Authentication documentation](https://docs.skyflow.com/docs/fundamentals/api-authentication). The bearer token can be a Skyflow API key (recommended for long-lived access), personal access token, or generated bearer token.

## Set Up Environment Variables

> **Note:** These instructions are for macOS. Windows and Linux users may need to adjust the shell configuration file and commands accordingly.

### Add the environment variables

Run these commands in Terminal, replacing the placeholder values with your actual Skyflow credentials:

```bash
echo 'export SKYFLOW_BEARER_TOKEN="your-token-here"' >> ~/.zshrc
echo 'export SKYFLOW_ACCOUNT_ID="your-account-id-here"' >> ~/.zshrc
```

### Apply the changes

**Quit Terminal completely** (Cmd + Q), then reopen it.

### Verify your setup

Run this command to confirm your environment variables are set (it checks that the value is present without printing the secret):

```bash
[ -n "$SKYFLOW_BEARER_TOKEN" ] && echo "SKYFLOW_BEARER_TOKEN is set" || echo "SKYFLOW_BEARER_TOKEN is not set"
```

If it prints `SKYFLOW_BEARER_TOKEN is set`, you're ready to restart Claude Code. If it prints `SKYFLOW_BEARER_TOKEN is not set`, revisit the steps above.

## Troubleshooting

**"Environment variable not set" errors:**

- Make sure you ran the `echo` commands to add the variables to `~/.zshrc`.
- Make sure you quit and reopened Terminal (not just opened a new tab).
- Make sure you restarted Claude Code.
