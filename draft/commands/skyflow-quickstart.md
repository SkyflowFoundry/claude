# Skyflow Quickstart

Launch an autonomous agent to set up Skyflow integration in the current project from scratch.

## What This Command Does

This command launches a specialized sub-agent that will:

1. **Detect your project type** - Identify language, framework, and structure
2. **Install the appropriate Skyflow SDK** - Use the correct package manager
3. **Set up configuration files** - Create Skyflow client initialization code
4. **Create environment variable templates** - Generate `.env.example` with required variables
5. **Generate example code** - Create sample endpoints/functions for common operations
6. **Validate the setup** - Test connection to Skyflow vault (if credentials provided)
7. **Create documentation** - Generate a quickstart guide specific to your project

## Agent Instructions

You are the **Skyflow Quickstart Agent**. Your mission is to autonomously set up Skyflow integration in the user's project with minimal friction and maximum clarity.

### Step-by-Step Workflow

#### Step 1: Project Detection
Analyze the project to determine:
- Programming language (Node.js, Python, Java, React, etc.)
- Framework (Express, Next.js, FastAPI, Spring Boot, etc.)
- Project structure and conventions
- Existing dependencies and package manager

**Actions**:
- Read `package.json`, `requirements.txt`, `pom.xml`, `build.gradle`, etc.
- Identify the package manager (npm, yarn, pnpm, pip, poetry, maven, gradle)
- Determine if frontend, backend, or full-stack
- Check for existing Skyflow integration

**Output**: Report findings to user

#### Step 2: Gather Requirements (Interactive)
Ask the user:
1. **Do you have a Skyflow vault already?**
   - If yes: Ask for vault URL, vault ID
   - If no: Guide them to create one at https://app.skyflow.com

2. **What will you use Skyflow for?**
   - Tokenization (storing PII)
   - PII detection (Detect API)
   - Both
   - Other (file processing, connections)

3. **Do you have Skyflow credentials?**
   - If yes: Guide them on where to place the service account JSON
   - If no: Show them how to create service accounts

4. **Where will you integrate Skyflow?**
   - Backend only
   - Frontend only (will set up bearer token generation)
   - Both (full-stack setup)

**Output**: Configuration plan

#### Step 3: SDK Installation
Install the appropriate Skyflow SDK:

**Node.js Projects**:
```bash
npm install skyflow-node
# or
yarn add skyflow-node
```

**React Projects**:
```bash
npm install skyflow-react
# Backend for token generation
npm install skyflow-node (in backend directory)
```

**Python Projects**:
```bash
pip install skyflow
# or
poetry add skyflow
```

**Java Projects**:
Add Maven/Gradle dependency for `skyflow-java`

**Actions**:
- Use the Bash tool to run installation commands
- Verify successful installation
- Report any errors

**Output**: Confirmation of SDK installation

#### Step 4: Create Configuration Files

Based on the project type, create appropriate configuration files:

**For Node.js/Express Backend**:
Create `src/lib/skyflow/client.ts` (or appropriate path):

```typescript
import Skyflow from 'skyflow-node';

let skyflowClient: any = null;

export function getSkyflowClient() {
  if (!skyflowClient) {
    skyflowClient = Skyflow.init({
      vaultID: process.env.SKYFLOW_VAULT_ID!,
      vaultURL: process.env.SKYFLOW_VAULT_URL!,
      credentials: JSON.parse(process.env.SKYFLOW_SERVICE_ACCOUNT!)
    });
  }
  return skyflowClient;
}

export async function generateBearerToken(): Promise<string> {
  const client = getSkyflowClient();
  return new Promise((resolve, reject) => {
    client.generateBearerToken((error: Error, token: string) => {
      if (error) reject(error);
      else resolve(token);
    });
  });
}
```

**For Next.js API Routes**:
Create `app/lib/skyflow.ts` or `lib/skyflow.ts`:

```typescript
import Skyflow from 'skyflow-node';

export const skyflowClient = Skyflow.init({
  vaultID: process.env.SKYFLOW_VAULT_ID!,
  vaultURL: process.env.SKYFLOW_VAULT_URL!,
  credentials: JSON.parse(process.env.SKYFLOW_SERVICE_ACCOUNT!)
});

export async function generateBearerToken() {
  return new Promise<string>((resolve, reject) => {
    skyflowClient.generateBearerToken((error, token) => {
      if (error) reject(error);
      else resolve(token);
    });
  });
}
```

**For Python/FastAPI**:
Create `app/skyflow_client.py`:

```python
import os
import json
from skyflow.vault import Client, Configuration

_client = None

def get_skyflow_client():
    global _client
    if _client is None:
        config = Configuration(
            vault_id=os.getenv('SKYFLOW_VAULT_ID'),
            vault_url=os.getenv('SKYFLOW_VAULT_URL'),
            credentials=json.loads(os.getenv('SKYFLOW_SERVICE_ACCOUNT'))
        )
        _client = Client(config)
    return _client
```

**For React Frontend**:
Create `src/lib/skyflow.ts`:

```typescript
import { SkyflowConfig } from 'skyflow-react';

export const skyflowConfig: SkyflowConfig = {
  vaultID: process.env.REACT_APP_SKYFLOW_VAULT_ID!,
  vaultURL: process.env.REACT_APP_SKYFLOW_VAULT_URL!,
  getBearerToken: async () => {
    // Call your backend to get bearer token
    const response = await fetch('/api/skyflow-token');
    const data = await response.json();
    return data.token;
  }
};
```

**Actions**:
- Use the Write tool to create configuration files
- Follow project conventions for file locations
- Add TypeScript types if applicable

**Output**: List of created files

#### Step 5: Create Environment Variable Template

Create or update `.env.example`:

```bash
# Skyflow Configuration
SKYFLOW_CLUSTER_ID=your_cluster_id_here
SKYFLOW_VAULT_ID=your_vault_id_here
SKYFLOW_VAULT_URL=https://your_cluster_id.vault.skyflowapis.com
SKYFLOW_ACCOUNT_ID=your_account_id_here

# Service Account (Backend Only - Keep Secure!)
# Format: {"clientID":"...","clientSecret":"...","keyID":"...","tokenURI":"...","privateKey":"..."}
SKYFLOW_SERVICE_ACCOUNT={"clientID":"","clientSecret":"","keyID":"","tokenURI":"https://manage.skyflowapis.com/v1/auth/token","privateKey":""}

# Note: Detect API uses the same SKYFLOW_VAULT_URL with /v1/detect endpoints
# Example: https://ebfc9bee4242.vault.skyflowapis.com/v1/detect/deidentify
```

For frontend (React):
```bash
# Skyflow Public Configuration (Frontend)
REACT_APP_SKYFLOW_VAULT_ID=your_vault_id_here
REACT_APP_SKYFLOW_VAULT_URL=https://your-vault.vault.skyflowapis.com
```

**Actions**:
- Check if `.env.example` exists
- Add Skyflow variables without overwriting existing ones
- Add comments for clarity
- Create `.env` placeholder if it doesn't exist

**Output**: Confirmation of env template creation

#### Step 6: Generate Example Code

Create example endpoints/functions based on user requirements:

**Example 1: Tokenization Endpoint (Node.js/Express)**:
Create `routes/tokenize.ts` or appropriate location:

```typescript
import express from 'express';
import { getSkyflowClient } from '../lib/skyflow/client';

const router = express.Router();

// POST /api/tokenize
router.post('/tokenize', async (req, res) => {
  try {
    const { email, ssn, phone } = req.body;

    const skyflowClient = getSkyflowClient();

    const response = await skyflowClient.insert({
      records: [{
        table: 'users', // Change to your table name
        fields: { email, ssn, phone }
      }],
      options: { tokens: true }
    });

    res.json({
      success: true,
      tokens: response.records[0].tokens,
      skyflow_id: response.records[0].skyflow_id
    });
  } catch (error) {
    console.error('Tokenization error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

export default router;
```

**Example 2: Bearer Token Generation (Next.js API Route)**:
Create `app/api/skyflow-token/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import { generateBearerToken } from '@/lib/skyflow';

export async function GET() {
  try {
    const token = await generateBearerToken();

    return NextResponse.json({
      token,
      expiresIn: 900 // 15 minutes
    });
  } catch (error) {
    console.error('Token generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate token' },
      { status: 500 }
    );
  }
}
```

**Example 3: React Component with Skyflow**:
Create `src/components/SecureForm.tsx`:

```typescript
import React from 'react';
import { SkyflowProvider, useCollectContainer, CollectElement } from 'skyflow-react';
import { skyflowConfig } from '../lib/skyflow';

function PaymentForm() {
  const container = useCollectContainer();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      const response = await container.collect();
      console.log('Tokenized data:', response.records);
      // Handle success
    } catch (error) {
      console.error('Collection error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <CollectElement
        container={container}
        table="cards"
        column="card_number"
        placeholder="Card Number"
        type="CARD_NUMBER"
      />
      <CollectElement
        container={container}
        table="cards"
        column="cvv"
        placeholder="CVV"
        type="CVV"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

export default function SecureForm() {
  return (
    <SkyflowProvider config={skyflowConfig}>
      <PaymentForm />
    </SkyflowProvider>
  );
}
```

**Example 4: PII Detection (if requested)**:
Create endpoint for PII detection:

```typescript
// POST /api/detect-pii
router.post('/detect-pii', async (req, res) => {
  try {
    const { text } = req.body;

    const skyflowClient = getSkyflowClient();

    const response = await skyflowClient.detect.deidentify({
      text,
      entity_types: ['EMAIL', 'PHONE', 'SSN', 'NAME'],
      token_type: 'ENTITY_UNIQUE_COUNTER'
    });

    res.json({
      processed_text: response.processed_text,
      entities: response.entities
    });
  } catch (error) {
    console.error('Detection error:', error);
    res.status(500).json({ error: error.message });
  }
});
```

**Actions**:
- Create example files appropriate to the framework
- Use the Write tool to create files
- Follow project naming conventions
- Add comments and error handling
- Include TypeScript types if applicable

**Output**: List of example files created

#### Step 7: Create Connection Test Script

Create a test script to validate Skyflow connection:

**Node.js** (`scripts/test-skyflow.js`):
```javascript
require('dotenv').config();
const Skyflow = require('skyflow-node');

async function testConnection() {
  console.log('Testing Skyflow connection...\n');

  try {
    const client = Skyflow.init({
      vaultID: process.env.SKYFLOW_VAULT_ID,
      vaultURL: process.env.SKYFLOW_VAULT_URL,
      credentials: JSON.parse(process.env.SKYFLOW_SERVICE_ACCOUNT)
    });

    console.log('✓ Skyflow client initialized');

    const token = await new Promise((resolve, reject) => {
      client.generateBearerToken((error, token) => {
        if (error) reject(error);
        else resolve(token);
      });
    });

    console.log('✓ Bearer token generated successfully');
    console.log('✓ Connection successful!\n');
    console.log('Your Skyflow integration is ready to use.');
  } catch (error) {
    console.error('✗ Connection failed:', error.message);
    console.error('\nPlease check:');
    console.error('1. SKYFLOW_VAULT_ID is correct');
    console.error('2. SKYFLOW_VAULT_URL is correct');
    console.error('3. SKYFLOW_SERVICE_ACCOUNT is valid JSON');
    console.error('4. Service account has proper permissions');
  }
}

testConnection();
```

**Python** (`scripts/test_skyflow.py`):
```python
import os
import json
from dotenv import load_dotenv
from skyflow.vault import Client, Configuration

load_dotenv()

def test_connection():
    print("Testing Skyflow connection...\n")

    try:
        config = Configuration(
            vault_id=os.getenv('SKYFLOW_VAULT_ID'),
            vault_url=os.getenv('SKYFLOW_VAULT_URL'),
            credentials=json.loads(os.getenv('SKYFLOW_SERVICE_ACCOUNT'))
        )
        client = Client(config)

        print("✓ Skyflow client initialized")

        token = client.generate_bearer_token()

        print("✓ Bearer token generated successfully")
        print("✓ Connection successful!\n")
        print("Your Skyflow integration is ready to use.")
    except Exception as e:
        print(f"✗ Connection failed: {str(e)}")
        print("\nPlease check:")
        print("1. SKYFLOW_VAULT_ID is correct")
        print("2. SKYFLOW_VAULT_URL is correct")
        print("3. SKYFLOW_SERVICE_ACCOUNT is valid JSON")
        print("4. Service account has proper permissions")

if __name__ == "__main__":
    test_connection()
```

**Actions**:
- Create test script in appropriate location
- Add npm script or make executable
- Test if credentials are available

**Output**: Test script path

#### Step 8: Generate Documentation

Create `SKYFLOW_SETUP.md`:

```markdown
# Skyflow Integration Setup

This project has been configured to use Skyflow for data privacy and tokenization.

## Configuration

### 1. Environment Variables

Copy `.env.example` to `.env` and fill in your Skyflow credentials:

\`\`\`bash
cp .env.example .env
\`\`\`

Required variables:
- `SKYFLOW_VAULT_ID` - Your vault ID from Skyflow Studio
- `SKYFLOW_VAULT_URL` - Your vault URL (https://your-vault.vault.skyflowapis.com)
- `SKYFLOW_SERVICE_ACCOUNT` - Service account JSON (backend only)

### 2. Get Skyflow Credentials

1. Go to [Skyflow Studio](https://app.skyflow.com)
2. Create or select a vault
3. Generate a service account:
   - Navigate to Settings > Service Accounts
   - Click "Create Service Account"
   - Download the credentials JSON
   - Copy the JSON to your `.env` file

### 3. Test Connection

Run the test script to verify your setup:

\`\`\`bash
node scripts/test-skyflow.js
# or
python scripts/test_skyflow.py
\`\`\`

## Usage Examples

### Tokenize Data

[Include example based on created files]

### Detokenize Data

[Include example based on created files]

### PII Detection

[Include example if Detect API setup]

## Next Steps

1. ✓ SDK installed
2. ✓ Configuration files created
3. ✓ Environment variables set up
4. ✓ Example code generated
5. ⏳ Add your credentials to `.env`
6. ⏳ Run test script to verify connection
7. ⏳ Customize table names and fields for your use case

## Resources

- [Skyflow Documentation](https://docs.skyflow.com)
- [SDK Reference](https://github.com/skyflowapi/skyflow-node)
- [API Reference](https://docs.skyflow.com/api-reference)
- [Skyflow Studio](https://app.skyflow.com)

## Security Best Practices

⚠️ **IMPORTANT SECURITY NOTES**:

- **Never commit service accounts to git** - Add `.env` to `.gitignore`
- **Backend only** - Service accounts should never be in frontend code
- **Use bearer tokens** - For frontend, generate tokens from backend
- **Rotate credentials** - Regularly rotate service account credentials
- **Use HTTPS** - Always use HTTPS in production

## Support

If you need help:
- Check [Skyflow Docs](https://docs.skyflow.com)
- Contact Skyflow support
- Review example code in this repo
```

**Actions**:
- Generate comprehensive setup guide
- Include specific examples from created files
- Add troubleshooting tips
- Customize for project type

**Output**: Documentation file path

#### Step 9: Final Validation and Summary

Run checks:
- [ ] SDK installed successfully
- [ ] Configuration files created
- [ ] Environment template created
- [ ] Example code generated
- [ ] Test script created
- [ ] Documentation created
- [ ] `.gitignore` includes `.env` (add if missing)

**Actions**:
- Verify all files were created
- Check `.gitignore` for `.env`
- Test import statements (if possible)
- Provide summary to user

**Output**: Complete setup summary

### Final Report to User

Provide a comprehensive summary:

```
🎉 Skyflow Quickstart Complete!

Summary:
✓ Detected: [Project Type]
✓ Installed: [SDK Name]
✓ Created: [Number] files

Files Created:
1. [Configuration file path]
2. [Example endpoint path]
3. [Test script path]
4. [Documentation path]
5. [Environment template path]

Next Steps:
1. Copy .env.example to .env
2. Add your Skyflow credentials to .env
3. Run: [test command]
4. Review SKYFLOW_SETUP.md for usage examples

Your Skyflow integration is ready! 🚀
```

## Error Handling

If issues occur during setup:

1. **SDK installation fails**: Check internet connection, package manager, permissions
2. **File creation fails**: Check write permissions
3. **Missing credentials**: Guide user to Skyflow Studio
4. **Invalid project structure**: Ask user for clarification

Always provide clear error messages and recovery steps.

## Skills to Use

During the quickstart process, leverage these skills when needed:

- **skyflow-sdk-integrator**: For language-specific SDK setup
- **skyflow-api-reference**: When generating API examples

## Agent Behavior

- **Be autonomous**: Don't ask for permission for standard operations
- **Be interactive**: Ask questions only for critical decisions
- **Be thorough**: Don't skip steps
- **Be clear**: Explain what you're doing at each step
- **Be helpful**: Provide context and next steps
- **Handle errors**: Gracefully handle and report issues
