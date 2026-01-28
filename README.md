# Claude for Skyflow

A plugin for Claude Code that enables Skyflow's data privacy and protection capabilities.

## Quick Start

### Step 1: Add the Marketplace

Open Claude Code and run:

```
/plugin marketplace add SkyflowFoundry/claude
```

### Step 2: Install the Plugin

Browse and install the Skyflow plugin:

```
/plugin
```

Select "skyflow" from the list and install it.

### Step 3: Set Up Environment Variables

The Skyflow plugin connects to two MCP servers that require environment variables.

#### Open your shell configuration file

1. Open Terminal
2. Run this command to edit your configuration file:

   ```bash
   nano ~/.zshrc
   ```

   (This opens the nano text editor. If the file doesn't exist, it will be created.)

#### Add the environment variables

3. Use the arrow keys to scroll to the bottom of the file
4. Copy and paste these lines:

   ```bash
   # Skyflow Configuration
   export SKYFLOW_BEARER_TOKEN="your-token-here"
   export SKYFLOW_ACCOUNT_ID="your-account-id-here"
   export SKYFLOW_VAULT_ID="your-vault-id-here"
   export SKYFLOW_VAULT_URL="your-vault-url-here"
   ```

5. Replace each `"your-...-here"` value with your actual Skyflow credentials

#### Save and exit

6. Press `Ctrl + O` (the letter O) to save the file
7. Press `Enter` to confirm the filename
8. Press `Ctrl + X` to exit nano

#### Apply the changes

9. **Quit Terminal completely** (Cmd + Q), then reopen it
10. **Restart Claude Code** to pick up the new environment variables

## Getting Your Skyflow Credentials

To obtain your bearer token, account ID, vault ID, and vault URL, see the [Skyflow API Authentication documentation](https://docs.skyflow.com/docs/fundamentals/api-authentication).

## Environment Variables Reference

| Variable                 | Description                            |
| ------------------------ | -------------------------------------- |
| `SKYFLOW_BEARER_TOKEN`   | Your Skyflow API authentication token  |
| `SKYFLOW_ACCOUNT_ID`     | Your Skyflow account identifier        |
| `SKYFLOW_VAULT_ID`       | Your vault identifier                  |
| `SKYFLOW_VAULT_URL`      | Your vault URL endpoint                |

These connect to:

- **Developer MCP** (`https://skyflow-mcp.dev`) - Uses account ID for development operations
- **Runtime MCP** (`https://www.pii-mcp.dev`) - Uses vault ID and URL for data operations

## Troubleshooting

**"Environment variable not set" errors:**

- Make sure you saved the `~/.zshrc` file after adding the variables
- Make sure you quit and reopened Terminal (not just opened a new tab)
- Make sure you restarted Claude Code

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
