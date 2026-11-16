# Skyflow Developer MCP

The Skyflow Developer MCP server (`https://mcp.skyflow.com`) for Claude Code. It provides access to Skyflow developer documentation, skills, and helpful resources for people integrating or implementing Skyflow. This is all most users need to get started.

The server uses OAuth so you sign in with your own Skyflow account. The plugin needs no environment variables and stores no credentials.

Part of the [Skyflow marketplace](../README.md).

## Before you install

Enable the MCP server in your Skyflow account first because sign-in fails until you do.

1. Sign in to Skyflow.
2. Open **AI Controls** in the left nav.
3. Enable the MCP server if it is not already enabled.
4. Add `https://claude.ai/api/mcp/auth_callback` and `https://claude.com/api/mcp/auth_callback` as redirect URLs.

## Install

Add the Skyflow marketplace:

```sh
/plugin marketplace add SkyflowFoundry/claude
```

Install the plugin:

```sh
/plugin install skyflow-developer-mcp@skyflow-marketplace
```

## Sign in

1. Run `/mcp` in Claude Code.
2. Select `skyflow-developer-mcp` and choose to authenticate.
3. Sign in to Skyflow in the browser that opens.

If the browser shows a connection error after you sign in then paste the full URL from the browser's address bar into the prompt in Claude Code.

## Upgrading from version 0.1

Version 0.1 pointed at the earlier proof-of-concept server and read `SKYFLOW_BEARER_TOKEN` and `SKYFLOW_ACCOUNT_ID`. Version 0.2 signs in with OAuth instead.

1. Run `/plugin marketplace update skyflow-marketplace` and then update the plugin.
2. Complete the steps in [Before you install](#before-you-install).
3. Sign in with `/mcp`.

You can keep the environment variables if the `skyflow-runtime-mcp` plugin uses them.

## Troubleshooting

**Sign-in fails with a redirect URI error:** Check that both Claude redirect URLs are listed under **AI Controls** in Skyflow.

**Sign-in fails before the browser opens:** Check that the MCP server is enabled under **AI Controls** in Skyflow.
