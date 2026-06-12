# Detect API (PII Detection & De-identification)

The Detect API automatically identifies and redacts PII in text and documents using ML-based entity detection.

**Base URL**: `https://{clusterID}.vault.skyflowapis.com/v1/detect`

**Authentication**: Bearer token required

**OpenAPI Spec**: See [detect.openapi.json](detect.openapi.json) for complete request/response schemas

---

## DEIDENTIFY TEXT - Detect and Replace PII

**Endpoint**: `POST /v1/detect/deidentify/string`
**Operation**: `deidentify_string`

Scans text for PII and replaces with tokens or placeholders.

```bash
curl -X POST "https://$CLUSTER_ID.vault.skyflowapis.com/v1/detect/deidentify/string" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Contact John at john@example.com or 555-123-4567",
    "entity_types": ["EMAIL", "PHONE_NUMBER", "NAME"],
    "token_type": "ENTITY_UNIQUE_COUNTER"
  }'
```

**Response**:
```json
{
  "processed_text": "Contact <NAME_1> at <EMAIL_1> or <PHONE_NUMBER_1>",
  "entities": [
    {
      "type": "NAME",
      "value": "John",
      "location": {"start": 8, "end": 12},
      "confidence": 0.95,
      "token": "<NAME_1>"
    },
    {
      "type": "EMAIL",
      "value": "john@example.com",
      "location": {"start": 16, "end": 32},
      "confidence": 0.99,
      "token": "<EMAIL_1>"
    }
  ]
}
```

**Token Types**:

| Type | Description | Example |
|------|-------------|---------|
| `ENTITY_ONLY` | Simple replacement | `<EMAIL>` |
| `ENTITY_UNIQUE_COUNTER` | With counter | `<EMAIL_1>`, `<EMAIL_2>` |
| `VAULT_TOKEN` | Stores in vault, returns Skyflow tokens | `tok_abc123` |

---

## REIDENTIFY TEXT - Restore Original PII

**Endpoint**: `POST /v1/detect/reidentify/string`
**Operation**: `reidentify_string`

Restores original values from de-identified text using entity mappings.

```bash
curl -X POST "https://$CLUSTER_ID.vault.skyflowapis.com/v1/detect/reidentify/string" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "processed_text": "Contact <NAME_1> at <EMAIL_1>",
    "entities": [
      {"type": "NAME", "token": "<NAME_1>", "value": "John"},
      {"type": "EMAIL", "token": "<EMAIL_1>", "value": "john@example.com"}
    ]
  }'
```

**Response**:
```json
{
  "text": "Contact John at john@example.com"
}
```

---

## DEIDENTIFY FILE - Process Documents

**Endpoint**: `POST /v1/detect/deidentify/file`
**Operation**: `deidentify_file`

Processes documents (PDF, images) to detect and redact PII. Returns an async run ID; poll `GET /v1/detect/runs/{run_id}` for the result.

```bash
# Submit file for processing
curl -X POST "https://$CLUSTER_ID.vault.skyflowapis.com/v1/detect/deidentify/file" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@document.pdf" \
  -F 'entity_types=["NAME","SSN","EMAIL"]' \
  -F "masking_method=REDACT" \
  -F "output_processed_image=true"
```

**Response** (async):
```json
{
  "request_id": "abc-123-def-456",
  "status": "PROCESSING"
}
```

**Check Status**:
```bash
curl -X GET "https://$CLUSTER_ID.vault.skyflowapis.com/v1/detect/runs/abc-123-def-456" \
  -H "Authorization: Bearer $TOKEN"
```

**Completed Response**:
```json
{
  "request_id": "abc-123-def-456",
  "status": "COMPLETED",
  "processed_file_url": "https://...",
  "entities": [
    {
      "type": "NAME",
      "value": "John Doe",
      "confidence": 0.95,
      "page": 1,
      "bounding_box": {...}
    }
  ]
}
```

**Form Fields**:
- `file`: Binary file content
- `entity_types`: JSON array of types to detect (optional, detects all if omitted)
- `masking_method`: `REDACT`, `MASK`, or `REPLACE` (default: `REDACT`)
- `output_processed_image`: Return redacted file (default: `true`)
- `output_ocr_text`: Return extracted text (default: `false`)

---

## Supported Entity Types

| Category | Entity Types |
|----------|--------------|
| **Personal** | `NAME`, `PERSON`, `DATE_OF_BIRTH`, `DOB`, `AGE`, `GENDER` |
| **Contact** | `EMAIL`, `EMAIL_ADDRESS`, `PHONE`, `PHONE_NUMBER`, `ADDRESS`, `STREET_ADDRESS` |
| **Government IDs** | `SSN`, `SOCIAL_SECURITY_NUMBER`, `DRIVER_LICENSE`, `DRIVERS_LICENSE_NUMBER`, `PASSPORT`, `PASSPORT_NUMBER` |
| **Financial** | `CREDIT_CARD`, `CREDIT_CARD_NUMBER`, `US_BANK_ACCOUNT_NUMBER`, `ROUTING_NUMBER` |
| **Network** | `IP_ADDRESS`, `IPV4`, `IPV6` |

See the OpenAPI spec for the complete list of 50+ supported entity types.
