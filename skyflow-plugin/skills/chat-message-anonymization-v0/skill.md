---
name: chat-message-anonymization-v0
description: integrate Skyflow's anonymization (de-identification and re-identification) capabilities into a Next.js AI chatbot application.
---


# Skyflow Chat Message Anonymization Integration

This document outlines the steps to integrate Skyflow's de-identification and re-identification capabilities into a Next.js AI chatbot application.

## Overview

The Skyflow integration provides automatic PII (Personally Identifiable Information) protection for chat messages:

- **User messages** are de-identified before database persistence and LLM processing
- **Assistant responses** are automatically re-identified after streaming completes
- **Manual toggle controls** allow users to reveal/hide PII in any message with a lock/unlock button
- **Visual feedback** shows tokens as blue badges and re-identified values as green badges to emphasize PII protection
- **Layout stability** prevents jarring size changes when toggling between tokenized and re-identified states

## Prerequisites

- Skyflow account with vault created (use Sky CLI or Skyflow dashboard)
- Skyflow vault credentials (VAULT_ID, VAULT_URL, VAULT_BEARER_TOKEN)
- Next.js application with Vercel AI SDK

## Step 1: Install Skyflow Node SDK

```bash
pnpm install skyflow-node
```

## Step 2: Configure Environment Variables

Add the following to your `.env.local` and `.env.example` files:

```bash
# Skyflow Configuration
VAULT_ID="your-vault-id"
VAULT_URL="https://your-cluster-id.vault.skyflowapis.com"
VAULT_BEARER_TOKEN="your-bearer-token"
WORKSPACE_ID="your-workspace-id"  # Optional
ACCOUNT_ID="your-account-id"      # Optional
```

**Important:** Use `VAULT_BEARER_TOKEN` (not `VAULT_KEY`) as the SDK expects bearer token authentication.

## Step 3: Create Skyflow Client Module

Create [lib/skyflow/client.ts](lib/skyflow/client.ts):

```typescript
import {
  Skyflow, SkyflowConfig, VaultConfig, Credentials,
  Env, LogLevel, DeidentifyTextRequest, ReidentifyTextRequest,
  DeidentifyTextOptions, TokenFormat, TokenType
} from 'skyflow-node';

let skyflowClient: Skyflow | null = null;

function getSkyflowClient(): Skyflow {
  // Validate environment variables
  const credentials: Credentials = {
    token: process.env.VAULT_BEARER_TOKEN!,
  };

  // Extract cluster ID from vault URL
  const vaultUrl = process.env.VAULT_URL!;
  const clusterIdMatch = vaultUrl.match(/https:\/\/(.+?)\.vault\.skyflowapis\.com/);
  const clusterId = clusterIdMatch[1];

  // Configure vault
  const vaultConfig: VaultConfig = {
    vaultId: process.env.VAULT_ID!,
    clusterId: clusterId,
    env: Env.PROD,
    credentials: credentials,
  };

  skyflowClient = new Skyflow({ vaultConfigs: [vaultConfig], ... });
  return skyflowClient;
}

export async function deidentifyText(text: string): Promise<string> {
  const client = getSkyflowClient();
  const detectService = client.detect();
  const request = new DeidentifyTextRequest(text);

  // CRITICAL: Configure vault token storage
  const options = new DeidentifyTextOptions();
  const tokenFormat = new TokenFormat();
  tokenFormat.setDefault(TokenType.VAULT_TOKEN);
  options.setTokenFormat(tokenFormat);

  const response = await detectService.deidentifyText(request, options);
  return response.processedText;
}

export async function reidentifyText(text: string): Promise<string> {
  const client = getSkyflowClient();
  const detectService = client.detect();
  const request = new ReidentifyTextRequest(text);
  const response = await detectService.reidentifyText(request);
  return response.processedText;
}

export function findEntityDifferences(
  deidentified: string,
  reidentified: string
): Array<{ start: number; end: number; token: string; value: string }> {
  // Use matchAll to safely find all tokens
  const tokenPattern = /\[([A-Z_]+_[a-zA-Z0-9]+)\]/g;
  const matches = Array.from(deidentified.matchAll(tokenPattern));

  // Map tokens to their reidentified positions for UI highlighting
  // ... implementation details in actual file
}
```

**Key Points:**

- Use `token` parameter (not `apiKey`) for bearer token authentication
- **Must set `TokenType.VAULT_TOKEN`** to store PII in vault for reidentification
- Use `matchAll` instead of regex `exec` loops to prevent infinite loops

## Step 4: Add Deidentification to Chat API

Modify [app/(chat)/api/chat/route.ts](app/(chat)/api/chat/route.ts):

```typescript
import { deidentifyText } from "@/lib/skyflow/client";

export async function POST(request: Request) {
  // ... auth and setup code

  // Deidentify user message before persisting and sending to LLM
  let deidentifiedParts;
  try {
    deidentifiedParts = await Promise.all(
      message.parts.map(async (part) => {
        if (part.type === "text") {
          const deidentifiedText = await deidentifyText(part.text);
          return { ...part, text: deidentifiedText };
        }
        return part;
      })
    );
  } catch (error) {
    console.error("Skyflow deidentification failed:", error);
    return new ChatSDKError(
      "bad_request:chat",
      "Failed to deidentify message. Message blocked for security."
    ).toResponse();
  }

  // Create deidentified message for storage and LLM
  const deidentifiedMessage = { ...message, parts: deidentifiedParts };

  // Use deidentified message for LLM conversation
  const uiMessages = [...convertToUIMessages(messagesFromDb), deidentifiedMessage];

  // Save deidentified message to database
  await saveMessages({
    messages: [{ chatId: id, id: deidentifiedMessage.id, role: "user", parts: deidentifiedMessage.parts, ... }],
  });

  // Continue with LLM streaming...
}
```

**Security Note:** Messages are blocked if deidentification fails, preventing PII from being stored or sent to LLMs.

## Step 5: Create Reidentification API Endpoint

Create [app/(chat)/api/reidentify/route.ts](app/(chat)/api/reidentify/route.ts):

```typescript
import { reidentifyText, findEntityDifferences } from "@/lib/skyflow/client";

export async function POST(request: Request) {
  // Authenticate user
  const session = await auth();
  if (!session?.user) {
    return new ChatSDKError("unauthorized:chat").toResponse();
  }

  const { chatId, messageId, text } = await request.json();

  // Verify user owns the chat
  const chat = await getChatById({ id: chatId });
  if (chat.userId !== session.user.id) {
    return new ChatSDKError("forbidden:chat").toResponse();
  }

  // Reidentify the text
  const reidentifiedText = await reidentifyText(text);

  // Find entity differences for highlighting
  const entities = findEntityDifferences(text, reidentifiedText);

  return Response.json({ text: reidentifiedText, entities, messageId });
}
```

## Step 6: Create Reidentifiable Text Component

Create [components/reidentifiable-text.tsx](components/reidentifiable-text.tsx):

```typescript
"use client";

import { motion } from "framer-motion";
import { Lock, Unlock } from "lucide-react";
import { useCallback, useEffect, useRef, useState } from "react";

export function ReidentifiableText({
  chatId,
  messageId,
  text,
  role,
  isLoading,
  onToggleButton  // Callback to provide toggle button to parent
}) {
  const [reidentifiedText, setReidentifiedText] = useState<string | null>(null);
  const [entities, setEntities] = useState<Entity[]>([]);
  const [showOriginal, setShowOriginal] = useState(false);
  const [minHeight, setMinHeight] = useState<number | null>(null);
  const contentRef = useRef<HTMLDivElement>(null);

  // Auto-reidentify assistant messages after streaming completes
  useEffect(() => {
    if (role === "assistant" && !isLoading && !reidentifiedText && !error) {
      const reidentify = async () => {
        const response = await fetch("/api/reidentify", {
          method: "POST",
          body: JSON.stringify({ chatId, messageId, text }),
        });
        const data = await response.json();
        setReidentifiedText(data.text);
        setEntities(data.entities);
        setShowOriginal(true);  // Auto-show for assistant messages
      };
      reidentify();
    }
  }, [chatId, messageId, text, role, isLoading, reidentifiedText, error]);

  // Manual toggle handler
  const handleToggle = useCallback(async () => {
    // Capture height to prevent layout shift
    if (contentRef.current && !minHeight) {
      setMinHeight(contentRef.current.offsetHeight);
    }

    // Fetch if not already fetched, otherwise just toggle
    if (!reidentifiedText && !isReidentifying) {
      // Fetch reidentified text...
      setShowOriginal(true);
    } else {
      setShowOriginal(!showOriginal);
    }
  }, [reidentifiedText, showOriginal]);

  // Provide toggle button to parent via callback
  useEffect(() => {
    const hasTokens = /\[[A-Z_]+_[a-zA-Z0-9]+\]/g.test(text);
    if (!hasTokens) {
      onToggleButton?.(null);
      return;
    }

    const button = (
      <button onClick={handleToggle} type="button">
        {showOriginal ? <Unlock size={14} /> : <Lock size={14} />}
      </button>
    );
    onToggleButton?.(button);
  }, [showOriginal, handleToggle]);

  // Render tokens as blue badges or reidentified values as green badges
  const content = showOriginal && reidentifiedText
    ? renderTextWithEntityHighlights(reidentifiedText, entities)
    : renderTextWithTokenStyling(text);

  return (
    <div ref={contentRef} style={minHeight ? { minHeight: `${minHeight}px` } : undefined}>
      {content}
    </div>
  );
}

function renderTextWithTokenStyling(text: string) {
  // Display tokens like [NAME_abc123] as prominent blue badges
  // with shadow, border, and high contrast
}

function renderTextWithEntityHighlights(text: string, entities: Entity[], shouldAnimate = false) {
  // Display reidentified values as prominent green badges
  // with shadow, border, and high contrast
  // Auto-reidentified assistant messages get pulse animation
  // Manually toggled messages show badges without animation
}
```

## Step 7: Update Message and MessageActions Components

Modify [components/message.tsx](components/message.tsx):

```typescript
import { ReidentifiableText } from "./reidentifiable-text";

const PurePreviewMessage = ({ chatId, message, ... }) => {
  const [reidentifyButton, setReidentifyButton] = useState<React.ReactNode>(null);

  // ...

  return (
    // In message content rendering:
    <ReidentifiableText
      chatId={chatId}
      messageId={message.id}
      text={part.text}
      role={message.role === "system" ? "assistant" : message.role}
      isLoading={isLoading}
      onToggleButton={setReidentifyButton}  // Receive button via callback
    />

    // In message actions:
    <MessageActions
      chatId={chatId}
      message={message}
      vote={vote}
      isLoading={isLoading}
      setMode={setMode}
      reidentifyButton={reidentifyButton}  // Pass button to actions
    />
  );
};
```

Modify [components/message-actions.tsx](components/message-actions.tsx):

```typescript
export function PureMessageActions({
  chatId,
  message,
  vote,
  isLoading,
  setMode,
  reidentifyButton  // Accept button from parent
}) {
  // ...

  return (
    <Actions>
      {reidentifyButton && (
        <div className="relative size-9 p-1.5">
          {reidentifyButton}
        </div>
      )}
      <Action onClick={handleCopy} tooltip="Copy">
        <CopyIcon />
      </Action>
      {/* ...other actions */}
    </Actions>
  );
}
```

## Message Flow

### User Message Flow

1. User types message with PII (e.g., "My SSN is 123-45-6789")
2. Frontend sends message to `/api/chat`
3. **Deidentification:** `deidentifyText()` converts to "My SSN is [SSN_abc123]"
4. Deidentified message saved to database
5. Deidentified message sent to LLM
6. If deidentification fails, message is blocked
7. **On reload/viewing old messages:**
   - User messages display with tokens as **blue badges**
   - Lock icon 🔒 button appears in message actions
   - User can click to reveal original values as **green badges**
   - User can toggle back and forth between tokenized and reidentified views

### Assistant Message Flow

1. LLM responds with message containing tokens (e.g., "Hello [NAME_xyz], your SSN [SSN_abc123] is protected")
2. Message streams to frontend with tokens visible as **blue badges**
3. Lock icon 🔒 button appears in message actions
4. After streaming completes (`isLoading` becomes false):
   - `useEffect` triggers automatic reidentification
   - Frontend calls `/api/reidentify` with message text
   - Backend reidentifies: "Hello John Smith, your SSN 123-45-6789 is protected"
   - UI updates with **green badges and pulse animation** highlighting reidentified entities
   - Button changes to unlock icon 🔓
5. **Manual toggle available:** User can click unlock 🔓 button to hide original values (show tokens) or lock 🔒 button to reveal them again

## Visual States

### 1. Deidentified State (Showing Tokens)

```
Hello [NAME_xyz]!  ← Prominent blue badge with shadow and border
      ^^^^^^^^^^^
[Lock icon 🔒] Copy ↑ ↓  ← Message actions with lock button
```

**Key characteristics:**

- Tokens displayed as blue badges with high visibility
- Lock icon 🔒 indicates values are hidden (tokenized)
- Click lock to reveal original values

### 2. Reidentified State (Showing Original Values)

```
Hello John Smith!  ← Prominent green badge with shadow and border
      ^^^^^^^^^^     (auto-reidentified messages pulse briefly)
[Unlock icon 🔓] Copy ↑ ↓  ← Message actions with unlock button
```

**Key characteristics:**

- Original values displayed as green badges with high visibility
- Unlock icon 🔓 indicates values are revealed
- Auto-reidentified assistant messages show pulse animation (fades over 2 seconds)
- Manually toggled messages show green badges without animation
- Click unlock to hide original values (return to tokens)

### 3. Toggle Behavior

**Without layout shift:**

- Component captures minimum height before first toggle
- Maintains minimum height to prevent jarring size changes
- Allows expansion if reidentified text is longer
- State does not persist across page reloads

## Key Implementation Details

### Critical Configuration Issues Solved

1. **Bearer Token Authentication:**
   - ❌ Wrong: `apiKey: process.env.VAULT_BEARER_TOKEN`
   - ✅ Correct: `token: process.env.VAULT_BEARER_TOKEN`

2. **Vault Token Storage:**
   - ❌ Wrong: No token format specified (PII not stored in vault)
   - ✅ Correct: `tokenFormat.setDefault(TokenType.VAULT_TOKEN)`

3. **Render Logic:**
   - ❌ Wrong: `if (!reidentifiedText || entities.length === 0)` (shows old text even when reidentified)
   - ✅ Correct: `if (!reidentifiedText)` then `if (entities.length > 0)` then fallback

4. **Entity Finding:**
   - ❌ Wrong: `while ((match = pattern.exec(...)))` (infinite loop risk)
   - ✅ Correct: `Array.from(text.matchAll(pattern))`

## Security Considerations

- User messages are blocked if deidentification fails
- Authorization checks ensure users can only reidentify their own messages
- Tokens are stored in Skyflow vault, not in application database
- Bearer tokens should be kept secure and rotated regularly
- Error logging only (no verbose PII logging in production)

## Files Modified

- [lib/skyflow/client.ts](lib/skyflow/client.ts) - Core Skyflow integration
- [app/(chat)/api/chat/route.ts](app/(chat)/api/chat/route.ts) - Deidentification on user messages
- [app/(chat)/api/reidentify/route.ts](app/(chat)/api/reidentify/route.ts) - Reidentification API
- [components/reidentifiable-text.tsx](components/reidentifiable-text.tsx) - Auto-reidentification and manual toggle UI
- [components/message.tsx](components/message.tsx) - Integration point and button state management
- [components/message-actions.tsx](components/message-actions.tsx) - Lock/unlock button rendering
- [.env.example](.env.example) - Environment variable documentation
- [.env.local](.env.local) - Local environment configuration

## Testing

### Initial Setup

1. Start development server: `pnpm dev`
2. Send a message with PII: "My name is John Smith and my SSN is 123-45-6789"
3. Verify message is deidentified in database: "My name is [NAME_xyz] and my SSN is [SSN_abc]"

### Auto-Reidentification (Assistant Messages)

1. Verify assistant response shows tokens as blue badges during streaming
2. Verify lock icon 🔒 appears in message actions
3. After streaming completes:
   - Tokens are replaced with original values shown as green badges
   - Green pulse animation highlights reidentified entities
   - Button changes to unlock icon 🔓

### Manual Toggle (All Messages)

1. **User messages:**
   - Reload the page
   - User messages display with blue token badges and lock icon 🔒
   - Click lock icon → original values appear as green badges, button becomes unlock icon 🔓
   - Click unlock icon → returns to showing blue token badges, button becomes lock icon 🔒

2. **Assistant messages:**
   - Click unlock icon 🔓 on an auto-reidentified message
   - Original values hide, tokens appear as blue badges
   - Button changes to lock icon 🔒
   - Click lock icon → reveals original values as green badges (no animation on manual toggle)

### Layout Stability

1. Toggle between tokenized and reidentified states multiple times
2. Verify message container maintains stable height (no jarring size changes)
3. Verify state resets on page reload (messages return to default state)

## Troubleshooting

**Issue:** Auth failure to Skyflow. Check and confirm what kind of credential is being used: API key - `apiKey`, Bearer Token - `token`, or others.

**Issue:** Reidentification returns same text as input

- **Cause:** Token format not configured to use vault tokens
- **Solution:** Ensure `tokenFormat.setDefault(TokenType.VAULT_TOKEN)` is set in deidentify options

**Issue:** UI not updating after reidentification

- **Cause:** Render logic checking `entities.length === 0`
- **Solution:** Check only `!reidentifiedText` before showing tokens

**Issue:** App freezing during reidentification

- **Cause:** Infinite loop in regex `exec()`
- **Solution:** Use `matchAll()` instead of `while (exec())`

## Resources

- [Skyflow Node SDK](https://github.com/skyflowapi/skyflow-node)
- [Skyflow Documentation](https://docs.skyflow.com/)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)


## Author notes

We can flesh this out with additional resources, e.g.:

my-skill/
├── SKILL.md (required)
├── reference.md (optional documentation)
├── examples.md (optional examples)
├── scripts/
│   └── helper.py (optional utility)
└── templates/
    └── template.txt (optional template)
