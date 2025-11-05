# Claude for Skyflow

A collection of skills, commands, subagents, prompts, etc for use with Claude Code when developing with Skyflow.

## Skills

Skills are the new Commands, complete with more context.

### Directory structure

my-skill/
├── SKILL.md (required)
├── reference.md (optional documentation)
├── examples.md (optional examples)
├── scripts/
│   └── helper.py (optional utility)
└── templates/
    └── template.txt (optional template)

### Sample

```md
---
name: chat-message-anonymization-v0
description: integrate Skyflow's deidentification and reidentification capabilities into a Next.js AI chatbot application.
---


# Skyflow Chat Message Anonymization Integration

## Overview

## Prerequisites

## Step 1: Install Skyflow Node SDK

...

## Key Implementation Details

### Critical Configuration Issues Solved

1. **Bearer Token Authentication:**
   - ❌ Wrong: `apiKey: process.env.VAULT_BEARER_TOKEN`
   - ✅ Correct: `token: process.env.VAULT_BEARER_TOKEN`

...

## Security Considerations

- User messages are blocked if deidentification fails

## Files Modified

- [lib/skyflow/client.ts](lib/skyflow/client.ts) - Core Skyflow integration

## Testing

### Initial Setup

1. Start development server: `pnpm dev`

## Troubleshooting

**Issue:** Auth failure to Skyflow. Check and confirm what kind of credential is being used: API key - `apiKey`, Bearer Token - `token`, or others.

**Issue:** Reidentification returns same text as input

- **Cause:** Token format not configured to use vault tokens
- **Solution:** Ensure `tokenFormat.setDefault(TokenType.VAULT_TOKEN)` is set in deidentify options

## Resources

- [Skyflow Node SDK](https://github.com/skyflowapi/skyflow-node)
- [Skyflow Documentation](https://docs.skyflow.com/)


```
