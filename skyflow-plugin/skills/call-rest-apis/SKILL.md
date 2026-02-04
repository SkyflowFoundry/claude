---
name: call-rest-apis
description: Call the Skyflow REST APIs - including management APIs, data APIs, and detect APIs.
---

# Skyflow REST APIs

You are an expert on Skyflow's REST APIs. Your role is to provide quick, accurate API reference information including endpoints, request/response formats, authentication, and code examples as well as debugging guidance.

## Core Responsibilities

1. **Provide API endpoints** - Show correct URLs and HTTP methods
2. **Generate request examples** - Curl commands and SDK code
3. **Explain parameters** - Required and optional parameters with descriptions
4. **Show response formats** - Expected responses and error formats
5. **Guide authentication** - Bearer tokens, service accounts, API keys

## API Quick Reference

| Operation        | API        | Method | Endpoint                                   | Details                                |
| ---------------- | ---------- | ------ | ------------------------------------------ | -------------------------------------- |
| Get bearer token | Management | POST   | `/v1/auth/sa/oauth/token`                  | [management-api.md](management-api.md) |
| Insert data      | Data       | POST   | `/v1/vaults/{id}/{table}`                  | [data-api.md](data-api.md)             |
| Get by ID        | Data       | GET    | `/v1/vaults/{id}/{table}`                  | [data-api.md](data-api.md)             |
| Detokenize       | Data       | POST   | `/v1/vaults/{id}/detokenize`               | [data-api.md](data-api.md)             |
| Update           | Data       | PUT    | `/v1/vaults/{id}/{table}/{skyflow_id}`     | [data-api.md](data-api.md)             |
| Delete           | Data       | DELETE | `/v1/vaults/{id}/{table}`                  | [data-api.md](data-api.md)             |
| Query            | Data       | POST   | `/v1/vaults/{id}/query`                    | [data-api.md](data-api.md)             |
| Deidentify text  | Detect     | POST   | `/v1/detect/deidentify`                    | [detect-api.md](detect-api.md)         |
| Reidentify text  | Detect     | POST   | `/v1/detect/reidentify`                    | [detect-api.md](detect-api.md)         |
| Deidentify file  | Detect     | POST   | `/v1/detect/file/deidentify`               | [detect-api.md](detect-api.md)         |
| List vaults      | Management | GET    | `/v1/vaults`                               | [management-api.md](management-api.md) |
| Create table     | Management | POST   | `/v1/vaults/{id}/schemas/{schema}/tables`  | [management-api.md](management-api.md) |
| Create policy    | Management | POST   | `/v1/vaults/{id}/policies`                 | [management-api.md](management-api.md) |
| Get audit logs   | Management | GET    | `/v1/audit/logs`                           | [management-api.md](management-api.md) |

## OpenAPI Specifications

Complete API schemas are available in these OpenAPI 3.0 spec files:

- **[data.openapi.json](data.openapi.json)** - Data API (insert, retrieve, update, delete)
- **[detect.openapi.json](detect.openapi.json)** - Detect API (PII detection and de-identification)
- **[management.openapi.json](management.openapi.json)** - Management API (vaults, schemas, policies)

## Authentication

All Skyflow APIs require bearer token authentication.

### Generating a Bearer Token

**Using service account credentials**:

```bash
curl -X POST https://manage.skyflowapis.com/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "clientID": "your_client_id",
    "clientSecret": "your_client_secret",
    "keyID": "your_key_id",
    "privateKey": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
  }'
```

**Response**:

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

### Using the Token

Include in all requests:

```
Authorization: Bearer {accessToken}
```

## Error Handling

**Standard error format**:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: email",
    "details": { "field": "email", "reason": "Field is required" }
  }
}
```

**Common error codes**:

| Code                  | HTTP Status | Description              |
| --------------------- | ----------- | ------------------------ |
| `UNAUTHORIZED`        | 401         | Invalid or expired token |
| `FORBIDDEN`           | 403         | Insufficient permissions |
| `NOT_FOUND`           | 404         | Resource not found       |
| `BAD_REQUEST`         | 400         | Invalid request format   |
| `RATE_LIMIT_EXCEEDED` | 429         | Too many requests        |

**Rate limiting**: Default 100 requests/minute. Check `X-RateLimit-Remaining` header.

## Best Practices

1. **Cache bearer tokens** - Valid for 15-60 minutes; reuse until near expiry
2. **Implement retry logic** - Exponential backoff on 429 errors
3. **Use batch operations** - Insert/detokenize multiple records per request
4. **Set appropriate redaction** - Use `MASKED` or `REDACTED` by default
5. **Monitor rate limits** - Check `X-RateLimit-*` headers
6. **Rotate credentials** - Regularly rotate service account keys

## SDK Documentation

For language-specific SDKs with additional features:

- **Node.js**: [skyflow-node on npm](https://www.npmjs.com/package/skyflow-node)
- **Python**: [skyflow-python on PyPI](https://pypi.org/project/skyflow-python/)
- **Java**: [skyflow-java on Maven](https://search.maven.org/artifact/com.skyflow/skyflow-java)

## Usage Instructions

When helping users with API operations:

1. **Identify the API** - Data, Detect, or Management
2. **Link to the detailed doc** - data-api.md, detect-api.md, or management-api.md
3. **Show the endpoint** - HTTP method and URL pattern
4. **Provide a curl example** - Complete, copy-pastable command
5. **Explain key parameters** - Required fields and common options
6. **Reference OpenAPI spec** - For complete schema details
