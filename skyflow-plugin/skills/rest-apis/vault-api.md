# Vault API (Data Operations)

The Vault API handles all data operations: inserting, retrieving, updating, and deleting sensitive data with automatic tokenization.

**Base URL**: `https://{vaultURL}/v1/vaults/{vaultID}`

**Authentication**: Bearer token required in all requests

**OpenAPI Spec**: See [data.openapi.json](data.openapi.json) for complete request/response schemas

## Common Headers

```
Authorization: Bearer {token}
Content-Type: application/json
X-Skyflow-Account-ID: {accountID}  # optional, for audit logs
```

---

## INSERT - Store and Tokenize Data

**Endpoint**: `POST /v1/vaults/{vaultID}/{tableName}`

Inserts records and returns tokens for sensitive fields.

```bash
curl -X POST "https://$VAULT_URL/v1/vaults/$VAULT_ID/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "records": [{
      "fields": {
        "email": "user@example.com",
        "ssn": "123-45-6789"
      }
    }],
    "tokenization": true
  }'
```

**Response**:
```json
{
  "records": [{
    "skyflow_id": "f8d8c7e4-63c7-4361-a89e-4e9a07e1ae9d",
    "tokens": {
      "email": "token_1234abcd",
      "ssn": "token_5678efgh"
    }
  }]
}
```

**Parameters**:
- `tokenization` (boolean): Return tokens for fields
- `continueOnError` (boolean): Continue processing on partial failures

---

## GET BY ID - Retrieve Data

**Endpoint**: `GET /v1/vaults/{vaultID}/{tableName}`

Retrieves records by Skyflow ID with configurable redaction.

```bash
curl -X GET "https://$VAULT_URL/v1/vaults/$VAULT_ID/users?skyflow_ids=id1,id2&redaction=MASKED" \
  -H "Authorization: Bearer $TOKEN"
```

**Response**:
```json
{
  "records": [{
    "fields": {
      "skyflow_id": "id1",
      "email": "user@example.com",
      "ssn": "XXX-XX-6789"
    }
  }]
}
```

**Query Parameters**:
- `skyflow_ids`: Comma-separated Skyflow IDs
- `redaction`: `PLAIN_TEXT`, `MASKED`, `REDACTED`, or `DEFAULT`
- `fields`: Comma-separated field names to retrieve

---

## DETOKENIZE - Retrieve Original Values

**Endpoint**: `POST /v1/vaults/{vaultID}/detokenize`

Converts tokens back to original values (requires permissions).

```bash
curl -X POST "https://$VAULT_URL/v1/vaults/$VAULT_ID/detokenize" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "detokenizationParameters": [
      {"token": "token_1234abcd", "redaction": "PLAIN_TEXT"}
    ]
  }'
```

**Response**:
```json
{
  "records": [{
    "token": "token_1234abcd",
    "value": "user@example.com"
  }]
}
```

---

## UPDATE - Modify Existing Records

**Endpoint**: `PUT /v1/vaults/{vaultID}/{tableName}`

Updates specific fields in existing records.

```bash
curl -X PUT "https://$VAULT_URL/v1/vaults/$VAULT_ID/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "records": [{
      "skyflow_id": "f8d8c7e4-63c7-4361-a89e-4e9a07e1ae9d",
      "fields": {
        "email": "newemail@example.com"
      }
    }]
  }'
```

---

## DELETE - Remove Records

**Endpoint**: `DELETE /v1/vaults/{vaultID}/{tableName}`

Permanently removes records from the vault.

```bash
curl -X DELETE "https://$VAULT_URL/v1/vaults/$VAULT_ID/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "records": [{
      "skyflow_id": "f8d8c7e4-63c7-4361-a89e-4e9a07e1ae9d"
    }]
  }'
```

---

## QUERY CONNECTION - Database Integrations

**Endpoint**: `POST /v1/vaults/{vaultID}/connections/{connectionID}/query`

Executes queries through configured database connections.

```bash
curl -X POST "https://$VAULT_URL/v1/vaults/$VAULT_ID/connections/$CONNECTION_ID/query" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT * FROM users WHERE user_id = ?",
    "parameters": ["user123"]
  }'
```

---

## Redaction Levels

| Level | Description |
|-------|-------------|
| `PLAIN_TEXT` | Unmasked data (requires permissions) |
| `MASKED` | Partially masked (e.g., `XXX-XX-6789`) |
| `REDACTED` | Fully redacted (e.g., `*********`) |
| `DEFAULT` | Uses field's default redaction policy |
