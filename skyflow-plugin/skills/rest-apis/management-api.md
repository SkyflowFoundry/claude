# Management API (Vault Administration)

The Management API handles vault administration: creating vaults, managing schemas, configuring policies, and accessing audit logs.

**Base URL**: `https://manage.skyflowapis.com/v1`

**Authentication**: Bearer token with management permissions

**OpenAPI Spec**: See [management.openapi.json](management.openapi.json) for complete request/response schemas

---

## LIST VAULTS

**Endpoint**: `GET /v1/vaults`

Returns all vaults accessible to the authenticated user.

```bash
curl -X GET "https://manage.skyflowapis.com/v1/vaults?limit=25&offset=0" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

**Response**:
```json
{
  "vaults": [
    {
      "ID": "vault_id_123",
      "name": "Production Vault",
      "url": "https://example.vault.skyflowapis.com",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 10
}
```

**Query Parameters**:
- `limit`: Results per page (default: 25, max: 50)
- `offset`: Pagination offset

---

## CREATE TABLE

**Endpoint**: `POST /v1/vaults/{vaultID}/schemas/{schemaName}/tables`

Adds a new table to an existing vault schema.

```bash
curl -X POST "https://manage.skyflowapis.com/v1/vaults/$VAULT_ID/schemas/default/tables" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "users",
    "fields": [
      {"name": "email", "type": "STRING", "mode": "REQUIRED"},
      {"name": "ssn", "type": "STRING", "mode": "NULLABLE"}
    ]
  }'
```

**Field Modes**:
- `REQUIRED`: Field cannot be null
- `NULLABLE`: Field can be null

---

## CREATE POLICY

**Endpoint**: `POST /v1/vaults/{vaultID}/policies`

Creates access control policies for vault data.

```bash
curl -X POST "https://manage.skyflowapis.com/v1/vaults/$VAULT_ID/policies" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "read_pii_policy",
    "description": "Allow reading PII with masking",
    "rules": [
      {
        "resource": "users.email",
        "action": "READ",
        "redaction": "MASKED"
      }
    ]
  }'
```

**Actions**: `READ`, `WRITE`, `DELETE`, `TOKENIZE`, `DETOKENIZE`

---

## GET AUDIT LOGS

**Endpoint**: `GET /v1/audit/logs`

Retrieves audit trail of vault operations.

```bash
curl -X GET "https://manage.skyflowapis.com/v1/audit/logs?start_time=2024-01-01T00:00:00Z&end_time=2024-01-31T23:59:59Z&limit=100" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

**Response**:
```json
{
  "logs": [
    {
      "timestamp": "2024-01-15T10:30:00Z",
      "action": "INSERT",
      "resource": "users",
      "user": "user@example.com",
      "status": "SUCCESS"
    }
  ]
}
```

**Query Parameters**:
- `start_time`: ISO 8601 timestamp
- `end_time`: ISO 8601 timestamp
- `limit`: Results per page
- `offset`: Pagination offset
