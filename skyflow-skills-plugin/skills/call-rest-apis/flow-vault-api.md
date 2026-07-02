# Flow Vault API

Flow Vault is Skyflow's high-performance vault product. It exposes a **V2** API surface for vault management, records, query, and tokenization, plus **V1** governance endpoints for policies, roles, service accounts, users, workspaces, and authentication.

**OpenAPI Spec**: See [flow-vault.openapi.json](flow-vault.openapi.json) for complete request/response schemas.

## Base URLs

Flow Vault uses two domains, depending on the operation:

| Operations                                                                                  | Environment | Base URL                                          |
| ------------------------------------------------------------------------------------------- | ----------- | ------------------------------------------------- |
| Records, Query, Tokens                                                                       | Production  | `https://{identifier}.skyvault.skyflowapis.com`   |
| Vault management, Policies, Roles, Service accounts, Authentication, Token exchange, Users, Workspaces | Production  | `https://{identifier}.skyflowapis.com`            |
| Records, Query, Tokens                                                                       | Sandbox     | `https://{identifier}.skyvault.skyflowapis-preview.com` |
| Vault management, Policies, Roles, Service accounts, Authentication, Token exchange, Users, Workspaces | Sandbox     | `https://{identifier}.skyflowapis-preview.com`    |

Replace `{identifier}` with your vault-specific identifier. In the examples below, `$VAULT_URL` refers to the appropriate base URL for the operation.

## Authentication

Every request requires **both** of these headers:

```text
Authorization: Bearer <your-bearer-token>
X-SKYFLOW-ACCOUNT-ID: <your-account-id>
```

`X-SKYFLOW-ACCOUNT-ID` is required by Flow Vault on all requests (unlike the classic Data API, where it's optional). See the [Authentication](#get-bearer-token) section below to generate a bearer token.

## Common Headers

```text
Authorization: Bearer $TOKEN
X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID
Content-Type: application/json
```

Set these environment variables to run the examples:

```bash
export VAULT_URL=$VAULT_URL
export ACCOUNT_ID=$ACCOUNT_ID
export TOKEN=$TOKEN
export VAULT_ID=$VAULT_ID
export TABLE_NAME=$TABLE_NAME
export WORKSPACE_ID=$WORKSPACE_ID
```

---

## Vault Management (V2)

### LIST VAULTS

**Endpoint**: `GET /v2/vaults`
**Operation**: `list-vaults`

Returns all vaults in your account.

```bash
curl -s -X GET "$VAULT_URL/v2/vaults" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

**Response**:

```json
{
  "vaults": [
    {
      "ID": "d408485953784308a000f8dcf81901ef",
      "name": "my_vault",
      "description": "Production data vault",
      "status": "ACTIVE"
    }
  ]
}
```

---

### CREATE VAULT

**Endpoint**: `POST /v2/vaults`
**Operation**: `create-vault`

Creates a vault from a `CreateVaultRequest` body. The `schema` object defines the vault structure.

```bash
curl -s -X POST "$VAULT_URL/v2/vaults" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my_vault",
    "description": "Production data vault for employee records",
    "workspaceID": "'"$WORKSPACE_ID"'",
    "schema": {
      "vaultType": "STRUCTURED",
      "tables": [
        { "name": "employees" }
      ],
      "columns": [
        { "name": "name", "tableName": "employees", "dataType": "STRING", "tokenGroups": ["det"] },
        { "name": "email", "tableName": "employees", "dataType": "STRING", "tokenGroups": ["det_rtf"] },
        { "name": "ssn", "tableName": "employees", "dataType": "STRING", "tokenGroups": ["det_reg"], "hashings": ["ssn_hash"] }
      ],
      "tokenGroup": [
        { "name": "det", "type": "DETERMINISTIC" },
        { "name": "det_rtf", "type": "DETERMINISTIC", "rightToForget": true },
        { "name": "det_reg", "type": "DETERMINISTIC", "format": "FPT", "regex": "^[A-Za-z0-9]{10}$" }
      ],
      "hashings": [
        { "name": "ssn_hash", "algorithm": "SHA256" }
      ]
    }
  }'
```

**Response**:

```json
{
  "ID": "d408485953784308a000f8dcf81901ef"
}
```

**Schema fields**:

- `schema.vaultType`\* (string): Vault type. Only `STRUCTURED` is supported.
- `schema.tables`\* (array): Each table requires a `name`. Optionally set `unique` column constraints and a `deleteTTL` (days) for automatic row expiration.
- `schema.columns`\* (array): Each column requires `name`, `tableName`, and `dataType` (`STRING`, `NUMBER`, `BOOL`, or `JSON`). Reference `tokenGroups` for tokenization and `hashings` for hashing.
- `schema.tokenGroup` (array): Each group requires `name` and `type` (`DETERMINISTIC` or `NONDETERMINISTIC`). Optional: `format` (`UUID` or `FPT`), `regex`, `transientTime` (TTL seconds), `rightToForget`.
- `schema.redactions` (array): Custom redaction patterns with `name`, `findPattern`, `replacePattern`.
- `schema.hashings` (array): Named hashing configs with `name` and `algorithm` (`SHA256`).
- `dryRun` (boolean): Validate the request without creating the vault. When `true`, the response includes `schemaValid` and `validationErrors`.

To enable SQL queries, set `queryService.enabled: true` in the schema.

---

### GET VAULT

**Endpoint**: `GET /v2/vaults/{ID}`
**Operation**: `get-vault`

Returns a vault's schema, status, and metadata.

```bash
curl -s -X GET "$VAULT_URL/v2/vaults/$VAULT_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

**Response**:

```json
{
  "vault": {
    "ID": "d408485953784308a000f8dcf81901ef",
    "name": "my_vault",
    "description": "Production data vault for employee records",
    "status": "ACTIVE"
  },
  "workspaceID": "z10198d5553411def9f2360c609gt3yx"
}
```

---

### UPDATE VAULT

**Endpoint**: `PATCH /v2/vaults/{ID}`
**Operation**: `update-vault`

Updates a vault's name, description, or schema.

```bash
curl -s -X PATCH "$VAULT_URL/v2/vaults/$VAULT_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Updated vault for employee and customer records",
    "schema": {
      "vaultType": "STRUCTURED",
      "tables": [
        { "name": "employees" },
        { "name": "customers" }
      ],
      "columns": [
        { "name": "name", "tableName": "employees", "dataType": "STRING" },
        { "name": "name", "tableName": "customers", "dataType": "STRING" }
      ]
    }
  }'
```

**Request Body**:

- `name`, `description`, `schema`: Fields to update.
- `dryRun` (boolean): Validate schema changes before applying them.
- `skipDataValidation` (boolean): Bypass data validation during the update.

---

### DELETE VAULT

**Endpoint**: `DELETE /v2/vaults/{ID}`
**Operation**: `delete-vault`

Permanently removes the vault, including all records and tokens. This cannot be undone.

```bash
curl -s -X DELETE "$VAULT_URL/v2/vaults/$VAULT_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

---

### VAULT METRICS

**Endpoint**: `POST /v2/vaults/metrics`
**Operation**: `flow-vault-metrics`

Returns record counts per table.

```bash
curl -s -X POST "$VAULT_URL/v2/vaults/metrics" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{ "vaultID": "'"$VAULT_ID"'" }'
```

**Response**:

```json
{
  "data": {
    "tables": {
      "employees": { "recordsCount": 1345 },
      "customers": { "recordsCount": 500 }
    }
  },
  "error": null
}
```

---

### UPLOAD SECRETS

**Endpoint**: `POST /v2/uploadSecrets`
**Operation**: `upload-vault-secrets`

Uploads a secret for each hashing configuration in the vault schema. `hashName` must exactly match a hashing config name. The secret acts as an HMAC key for hash computation; an empty string is valid. Columns with hashing don't return hash values until secrets are uploaded.

```bash
curl -s -X POST "$VAULT_URL/v2/uploadSecrets" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "secrets": [
      { "hashName": "ssn_hash", "secret": "my-secret-value" }
    ]
  }'
```

**Response**:

```json
{ "vaultID": "d408485953784308a000f8dcf81901ef" }
```

---

## Records (V2)

Records are the primary data units in Flow Vault. Responses return tokens alongside data based on the vault's token group configuration, and a `hashedData` field when columns have hashing configured. Batch operations support **partial success**: each record in the response has its own `httpCode` and `error` field.

### INSERT RECORDS

**Endpoint**: `POST /v2/records/insert`
**Operation**: `insert-records`

Inserts one or more records. `tableName` can be set at the request level or per-record (per-record takes precedence).

```bash
curl -s -X POST "$VAULT_URL/v2/records/insert" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "tableName": "'"$TABLE_NAME"'",
    "records": [
      { "data": { "name": "Jane Doe", "email": "jane.doe@example.com", "ssn": "123-45-6789" } }
    ]
  }'
```

**Response**:

```json
{
  "records": [
    {
      "skyflowID": "97cdd1af-02ac-47eb-ab0d-8339dbef6ccb",
      "tokens": {
        "email": [{ "token": "6a37c40a-7e4a-4f6b-b202-a84ca6a5857e", "tokenGroupName": "det_rtf" }],
        "ssn": [{ "token": "AMmmtFZyRO", "tokenGroupName": "det_reg" }]
      },
      "error": null,
      "httpCode": 200
    }
  ]
}
```

**Upsert**: To insert-or-update by matching unique column values, include an `upsert` object per record with `updateType` and `uniqueColumns`:

```json
"upsert": { "updateType": "UPDATE", "uniqueColumns": ["email"] }
```

---

### GET RECORDS

**Endpoint**: `POST /v2/records/get`
**Operation**: `get-records`

Retrieves records by `skyflowIDs` **or** `uniqueValues` (not both). Use `columnRedactions` to set the redaction per column, `columns` to limit returned columns, and `limit`/`offset` for pagination.

```bash
curl -s -X POST "$VAULT_URL/v2/records/get" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "tableName": "'"$TABLE_NAME"'",
    "skyflowIDs": ["97cdd1af-02ac-47eb-ab0d-8339dbef6ccb"],
    "columnRedactions": [
      { "columnName": "name", "redaction": "plain_text" },
      { "columnName": "ssn", "redaction": "redacted" }
    ]
  }'
```

**Response**:

```json
{
  "records": [
    {
      "skyflowID": "97cdd1af-02ac-47eb-ab0d-8339dbef6ccb",
      "data": { "name": "Jane Doe", "ssn": "XXX-XX-6789" },
      "error": null,
      "httpCode": 200
    }
  ]
}
```

Get by unique values instead of Skyflow IDs:

```json
"uniqueValues": [ { "data": { "email": "jane.doe@example.com" } } ]
```

---

### UPDATE RECORDS

**Endpoint**: `POST /v2/records/update`
**Operation**: `update-records`

Updates records by `skyflowID`. The `updateType` controls behavior:

- `UPDATE`: Merges provided data with existing data (fields not sent are preserved).
- `REPLACE`: Replaces the entire record (fields not sent are removed).

```bash
curl -s -X POST "$VAULT_URL/v2/records/update" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "tableName": "'"$TABLE_NAME"'",
    "records": [
      { "skyflowID": "97cdd1af-02ac-47eb-ab0d-8339dbef6ccb", "data": { "email": "jane.updated@example.com" } }
    ],
    "updateType": "UPDATE"
  }'
```

---

### DELETE RECORDS

**Endpoint**: `POST /v2/records/delete`
**Operation**: `delete-records`

Deletes records by `skyflowIDs` or `uniqueValues`.

```bash
curl -s -X POST "$VAULT_URL/v2/records/delete" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "tableName": "'"$TABLE_NAME"'",
    "skyflowIDs": ["97cdd1af-02ac-47eb-ab0d-8339dbef6ccb"]
  }'
```

---

## Query (V2)

### EXECUTE QUERY

**Endpoint**: `POST /v2/query`
**Operation**: `execute-query`

Runs a SQL `SELECT` query against vault data. Requires `queryService.enabled: true` on the vault.

```bash
curl -s -X POST "$VAULT_URL/v2/query" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "query": "SELECT name, email FROM employees WHERE age > 21 LIMIT 10"
  }'
```

**Response**:

```json
{
  "records": [
    { "data": { "name": "Jane Doe", "email": "jane.doe@example.com" } }
  ]
}
```

**Supported SQL**:

- Clauses: `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`, `LIMIT`, `OFFSET`. `INSERT`/`UPDATE`/`DELETE` are **not** supported.
- Functions: `AVG`, `COUNT`, `MAX`, `MIN`, `SUM`, and `REDACT(column, 'redaction')` to apply redaction to column values.

**Constraints**: Maximum 25 records per response. Token and file URL values aren't returned. Use `LIMIT`/`OFFSET` to paginate.

---

## Tokens (V2)

### GET TOKENS

**Endpoint**: `POST /v2/records/getTokens`
**Operation**: `getTokens`

Retrieves the deterministic token previously issued for a plaintext value in a token group, enabling token-based lookups without storing raw PII. Only **deterministic** token groups are supported. Requires `TOKENLOOKUP` permission on each referenced token group.

```bash
curl -s -X POST "$VAULT_URL/v2/records/getTokens" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "records": [
      { "value": "9988998899", "tokenGroupName": "phoneNumberGroup" },
      { "value": "john@example.com", "tokenGroupName": "emailGroup" }
    ]
  }'
```

**Response**:

```json
{
  "records": [
    { "token": "3232-4444-3232-1234", "tokenGroupName": "phoneNumberGroup", "value": "9988998899", "error": null, "httpCode": 200 }
  ]
}
```

When some records succeed and others fail, the API returns HTTP 207 (Partial Success); a per-record HTTP 404 indicates the value doesn't exist in the token group.

---

### DETOKENIZE

**Endpoint**: `POST /v2/tokens/detokenize`
**Operation**: `detokenize`

Converts tokens back to original values. Use `tokenGroupRedactions` to control the returned redaction per token group.

```bash
curl -s -X POST "$VAULT_URL/v2/tokens/detokenize" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "vaultID": "'"$VAULT_ID"'",
    "tokens": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    "tokenGroupRedactions": [
      { "tokenGroupName": "deterministic_group", "redaction": "plain_text" }
    ]
  }'
```

**Response**:

```json
{
  "response": [
    { "token": "a1b2c3d4-e5f6-7890-abcd-ef1234567890", "value": "123-45-6789", "tokenGroupName": "deterministic_group", "error": null, "httpCode": 200 }
  ]
}
```

---

## Policies (V1)

Flow Vault uses a policy-based access control (PBAC) model. Each policy contains rules at the column, table, or token group level. Each rule has `actions`, an `effect` (`ALLOW` or `DENY`), an optional `redaction`, and an optional row filter.

**Policy operations**: `ALL`, `CREATE`, `READ`, `UPDATE`, `DELETE`, `TOKENLOOKUP`, `DETOKENIZATION`.

### LIST POLICIES

**Endpoint**: `GET /v1/policies`
**Operation**: `list-policies`

```bash
curl -s -X GET "$VAULT_URL/v1/policies?resource.ID=$VAULT_ID&resource.type=VAULT" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

**Query Parameters**:

- `resource.ID`: The resource (for example, vault) ID.
- `resource.type`: The resource type (for example, `VAULT`).

---

### CREATE POLICY

**Endpoint**: `POST /v1/policies`
**Operation**: `create-policy`

```bash
curl -s -X POST "$VAULT_URL/v1/policies" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "read_pii_plaintext",
    "displayName": "Read PII Plain Text",
    "description": "Allows reading PII columns in plain text",
    "resource": { "ID": "'"$VAULT_ID"'", "type": "VAULT" },
    "ruleParams": [
      {
        "name": "read_employee_names",
        "columnRuleParams": {
          "vaultID": "'"$VAULT_ID"'",
          "columns": ["employees.name", "employees.email"],
          "actions": ["READ"],
          "effect": "ALLOW",
          "redaction": "plain_text"
        }
      }
    ],
    "activated": true
  }'
```

**Response**:

```json
{ "ID": "p3a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5" }
```

Rules can use `columnRuleParams`, `tableRuleParams`, or token group rules. Set `activated` to `false` (the default) to create a disabled policy.

---

### GET / UPDATE / DELETE POLICY

- **Get**: `GET /v1/policies/{ID}` (`get-policy`)
- **Update**: `PATCH /v1/policies/{ID}` (`update-policy`) — update `name`, `description`, `ruleParams`, or `activated`.
- **Delete**: `DELETE /v1/policies/{ID}` (`delete-policy`)

### ASSIGN / UNASSIGN POLICY

**Endpoint**: `POST /v1/policies/assign` (`assign-policy`) and `POST /v1/policies/unassign` (`unassign-policy`)

Assign a policy to roles and/or members. Use `exceptions` to exclude specific members from a role-based assignment.

```bash
curl -s -X POST "$VAULT_URL/v1/policies/assign" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "ID": "'"$POLICY_ID"'",
    "roleIDs": ["r1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6"]
  }'
```

---

## Roles (V1)

Roles group permissions and policies that you assign to users and service accounts. A role's `type` is either `SYSTEM` (Skyflow-defined, immutable) or `CUSTOM` (user-defined). Only `CUSTOM` roles can be updated or deleted.

### LIST ROLES

**Endpoint**: `GET /v1/roles`
**Operation**: `list-roles`

```bash
curl -s -X GET "$VAULT_URL/v1/roles?resource.ID=$VAULT_ID&resource.type=VAULT" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID"
```

---

### CREATE ROLE

**Endpoint**: `POST /v1/roles`
**Operation**: `create-role`

```bash
curl -s -X POST "$VAULT_URL/v1/roles" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "roleDefinition": {
      "name": "data_analyst",
      "displayName": "Data Analyst",
      "description": "Read-only access to vault data with redaction",
      "permissions": ["READ"],
      "levels": ["VAULT"]
    },
    "resource": { "ID": "'"$VAULT_ID"'" }
  }'
```

**Response**:

```json
{ "ID": "r1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6" }
```

---

### GET / UPDATE / DELETE ROLE

- **Get**: `GET /v1/roles/{ID}` (`get-role`)
- **Update**: `PATCH /v1/roles/{ID}` (`update-role`) — update the `roleDefinition`.
- **Delete**: `DELETE /v1/roles/{ID}` (`delete-role`) — `CUSTOM` roles only.

### ASSIGN / UNASSIGN ROLE

**Endpoint**: `POST /v1/roles/assign` (`assign-role`) and `POST /v1/roles/unassign` (`unassign-role`)

Assign a role to members. Optionally include a `condition` expression that restricts when the role applies.

```bash
curl -s -X POST "$VAULT_URL/v1/roles/assign" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "ID": "'"$ROLE_ID"'",
    "members": [ { "ID": "m4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9" } ]
  }'
```

---

## Service Accounts (V1)

Service accounts are machine identities that applications use to authenticate. Each has a key pair for generating JWT tokens. Optional controls: `enforceContextID` (require a `ctx` claim in JWT assertions) and `enforceSignedDataTokens` (require data tokens to be signed).

### CREATE SERVICE ACCOUNT

**Endpoint**: `POST /v1/serviceAccounts`
**Operation**: `create-service-account`

```bash
curl -s -X POST "$VAULT_URL/v1/serviceAccounts" \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-SKYFLOW-ACCOUNT-ID: $ACCOUNT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "serviceAccount": {},
    "clientConfiguration": { "enforceContextID": false, "enforceSignedDataTokens": false },
    "accountID": "'"$ACCOUNT_ID"'"
  }'
```

**Response** (credentials are returned **only once** — store `privateKey` and `apiKey` securely):

```json
{
  "clientID": "sa1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6",
  "keyID": "k1a2b3c4d5e6f7a8",
  "privateKey": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
  "apiKeyID": "ak1a2b3c4d5e6f7a8",
  "apiKey": "sky-ab12c-d3ef4567890abcdef1234567890abcdef",
  "keyAlgorithm": "KEY_ALG_RSA_2048"
}
```

**Other operations**:

- List: `GET /v1/serviceAccounts` (`list-service-accounts`)
- Get: `GET /v1/serviceAccounts/{ID}` (`get-service-account`)
- Update: `PATCH /v1/serviceAccounts/{ID}` (`update-service-account`)
- Delete: `DELETE /v1/serviceAccounts/{ID}` (`delete-service-account`)
- API keys: `.../apikey` (`create-api-key`, `list-api-keys`, `get-api-key`, `delete-api-key`, `rotate-api-key`)
- Keys: `.../keys` (`create-service-account-key`, `list-service-account-keys`, `get-service-account-key`, `delete-service-account-key`, `rotate-service-account-key`)
- Signed token keys: `.../signedtokenkey` (`create-signed-data-token-key`, `list-signed-data-token-keys`, `get-signed-data-token-key`, `delete-signed-data-token-key`, `rotate-signed-data-token-key`)

---

## Authentication (V1)

### GET BEARER TOKEN

**Endpoint**: `POST /v1/auth/sa/oauth/token`
**Operation**: `get-bearer-token`

Exchanges a signed JWT assertion (built from your service account credentials) for a bearer token. This endpoint does not require an existing `Authorization` header.

```bash
curl -s -X POST "$VAULT_URL/v1/auth/sa/oauth/token" \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type": "urn:ietf:params:oauth:grant-type:jwt-bearer",
    "assertion": "<your-signed-jwt>"
  }'
```

**Response**:

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer"
}
```

**Request Body**:

- `grant_type`\*: Must be `urn:ietf:params:oauth:grant-type:jwt-bearer`.
- `assertion`\*: A signed JWT with claims `iss` (issuer), `key` (key ID), `aud` (recipient), `exp` (expiry), `sub` (subject), and optional `ctx` (context for context-aware authorization).
- `scope`: Subset of roles: `"role:<roleID1> role:<roleID2>"`.

**Token exchange (STS)**: For delegation/impersonation flows, use `POST /v1/auth/sts/token` (`get-sts-token`) and manage configs via `/v1/sts/config` (`create-sts-config`, `list-sts-configs`, `get-sts-config`, `update-sts-config`, `delete-sts-config`).

---

## Users and Workspaces (V1)

- **Users**: `/v1/users` — `create-user`, `list-users`, `get-user`, `update-user`, `delete-user`.
- **Workspaces**: `/v1/workspaces` — `create-workspace`, `list-workspaces`, `get-workspace`, `update-workspace`, `delete-workspace`, and `list-workspace-members` (`GET /v1/workspaces/{ID}/members`).
- **Accounts / Resources**: `/v1/accounts` (`list-accounts`, `get-account`, `update-account`, `list-regions`) and `/v1/resources` (`list-resources`).

---

## Redaction Levels

Flow Vault redaction values are lowercase (unlike the classic Data API):

| Level                 | Description                                                             |
| --------------------- | ----------------------------------------------------------------------- |
| `plain_text`          | Returns the full original value (requires permissions)                  |
| `redacted`            | Returns a fully redacted placeholder                                    |
| Custom redaction name | Returns the value per a user-defined redaction in the schema (e.g. `emailMask`) |

If no redaction is specified, Flow Vault applies the default configured for the column or token group.

## Token Groups

Token groups define tokenization behavior. Each has a `type`:

- `DETERMINISTIC`: Same input always yields the same token. Required for standalone `getTokens`.
- `NONDETERMINISTIC`: Same input can yield different tokens.

Optional properties: `format` (`UUID` or `FPT` for format-preserving tokens), `regex` (with `format: FPT`), `transientTime` (token TTL in seconds, `NONDETERMINISTIC` only), and `rightToForget` (when `true` on `DETERMINISTIC` tokens, deleting the last record with a value invalidates its tokens for detokenization).

## Hashings

Hashings compute deterministic cryptographic fingerprints (`SHA256`) of column data during Insert, Get, and Update. Hashes are **not stored** — they're recomputed each time. Configure them in the vault `schema.hashings`, subscribe columns via `columns[].hashings`, and upload one secret per hashing config with `POST /v2/uploadSecrets`. Responses then include a `hashedData` map (column → array of `{ hashName, data }`).

## Error Handling

Batch operations (records, tokens) support **partial success**. Inspect the per-item `httpCode` and `error` fields to identify which items succeeded and which failed. A batch with mixed results returns HTTP 207.

```json
{
  "records": [
    { "skyflowID": "97cdd1af-02ac-47eb-ab0d-8339dbef6ccb", "error": null, "httpCode": 200 },
    { "skyflowID": "", "error": "Database rate limit exceeded.", "httpCode": 429 }
  ]
}
```
