# Skyflow Skills

Guided Skyflow workflows for Claude Code. This plugin packages **skills** — structured documentation, sample schemas, and API examples that Claude can reference when helping you implement Skyflow features.

This plugin contains skills only. It does **not** connect to any MCP server and requires **no environment variables**. For live API access, pair it with the [`skyflow-developer-mcp`](../skyflow-developer-mcp-plugin/README.md) plugin.

Part of the [Skyflow marketplace](../README.md).

## Install

```sh
/plugin marketplace add SkyflowFoundry/claude
/plugin install skyflow-skills@skyflow-marketplace
```

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

## Learn More

For complete documentation on Claude Code plugins, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).
