# Error Handling & Error Payloads

## Introduction

Error handling defines how an API communicates failure to its consumers. A well-designed error payload gives the consumer enough information to understand what went wrong, why, and what to do next — without exposing internal system details.

This section covers HTTP status code selection, structured error payload design, RFC 7807 (Problem Details), and enterprise error taxonomy.

---

## Why it matters

Poor error responses force consumers to guess:
- Is `500` a transient error I should retry, or a bug I should report?
- What field failed validation? Which rule?
- Is this error the API's fault or mine?

Good error responses are self-describing contracts that help consumers build resilient integrations.

---

## Prerequisites

- HTTP basics (status codes, headers, methods)
- JSON fundamentals

---

## Architecture

### HTTP Status Code Groups

```
1xx  Informational   ← rarely used in REST APIs
2xx  Success
3xx  Redirection     ← handled by gateway, not your API
4xx  Client errors   ← consumer did something wrong
5xx  Server errors   ← your API/system did something wrong
```

### Error payload anatomy

```json
{
    "type":       "https://api.example.com/errors/validation-error",
    "title":      "Validation Error",
    "status":     400,
    "detail":     "The 'amount' field must be a positive number.",
    "instance":   "/v1/payments/TXN-2025-001",
    "errorCode":  "VALIDATION_001",
    "correlationId": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp":  "2025-06-25T10:30:00Z",
    "errors": [
        {
            "field":   "amount",
            "message": "Must be greater than 0",
            "value":   -500
        }
    ]
}
```

---

## Implementation

### HTTP Status Codes — Enterprise Reference

#### 2xx Success

| Code | Name | When to use |
|---|---|---|
| `200` | OK | Successful GET, PUT, PATCH |
| `201` | Created | Successful POST that created a resource |
| `202` | Accepted | Async operation accepted, processing later |
| `204` | No Content | Successful DELETE or action with no response body |

#### 4xx Client Errors

| Code | Name | When to use |
|---|---|---|
| `400` | Bad Request | Malformed JSON, invalid field type, missing required field |
| `401` | Unauthorized | Missing or invalid authentication token |
| `403` | Forbidden | Authenticated but not authorised for this resource |
| `404` | Not Found | Resource does not exist |
| `405` | Method Not Allowed | HTTP method not supported on this endpoint |
| `409` | Conflict | Duplicate resource, state conflict |
| `410` | Gone | Resource existed but was permanently deleted |
| `422` | Unprocessable Entity | Valid JSON, but fails business validation rules |
| `429` | Too Many Requests | Rate limit exceeded |

#### 5xx Server Errors

| Code | Name | When to use |
|---|---|---|
| `500` | Internal Server Error | Unexpected server error (non-retriable) |
| `502` | Bad Gateway | Upstream/downstream system returned invalid response |
| `503` | Service Unavailable | API temporarily down, under maintenance |
| `504` | Gateway Timeout | Upstream system did not respond in time |

### Error Payload Standard (RFC 7807 — Problem Details)

```json
{
    "type":       "URI identifying the error type (link to docs)",
    "title":      "Short human-readable summary",
    "status":     400,
    "detail":     "Longer explanation of this specific occurrence",
    "instance":   "URI of the specific request that caused the error",
    "errorCode":  "Machine-readable code for programmatic handling",
    "correlationId": "Trace ID for log correlation",
    "timestamp":  "ISO 8601 timestamp"
}
```

### Enterprise Error Taxonomy

```
ERROR CATEGORIES
├── VALIDATION_ERROR     → 400 | Consumer sent invalid input
├── AUTHENTICATION_ERROR → 401 | Token missing, expired, or malformed
├── AUTHORIZATION_ERROR  → 403 | Token valid but access denied
├── NOT_FOUND_ERROR      → 404 | Resource does not exist
├── CONFLICT_ERROR       → 409 | Duplicate or state conflict
├── BUSINESS_RULE_ERROR  → 422 | Valid input, fails business logic
├── RATE_LIMIT_ERROR     → 429 | Too many requests
├── DOWNSTREAM_ERROR     → 502 | Upstream system failure
├── TIMEOUT_ERROR        → 504 | Upstream system timed out
└── INTERNAL_ERROR       → 500 | Unexpected server failure
```

---

## Examples

### Validation error (400) — multiple field failures

```json
{
    "type": "https://api.acme.com/errors/validation-error",
    "title": "Validation Error",
    "status": 400,
    "detail": "One or more fields failed validation.",
    "errorCode": "VALIDATION_ERROR",
    "correlationId": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2025-06-25T10:30:00Z",
    "errors": [
        { "field": "amount",   "message": "Must be greater than 0",      "value": -500 },
        { "field": "currency", "message": "Must be a valid ISO currency code", "value": "KSH" }
    ]
}
```

### Not found error (404)

```json
{
    "type": "https://api.acme.com/errors/not-found",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "Customer with ID 'C999' does not exist.",
    "errorCode": "NOT_FOUND_ERROR",
    "correlationId": "7b4e9120-f38a-42bc-8b11-aa3344556677",
    "timestamp": "2025-06-25T10:31:00Z"
}
```

### Rate limit error (429) with retry guidance

```json
{
    "type": "https://api.acme.com/errors/rate-limit",
    "title": "Rate Limit Exceeded",
    "status": 429,
    "detail": "You have exceeded 100 requests per minute. Retry after 45 seconds.",
    "errorCode": "RATE_LIMIT_ERROR",
    "correlationId": "aa1122bb-cc33-44dd-ee55-ff6677889900",
    "timestamp": "2025-06-25T10:32:00Z",
    "retryAfter": 45
}
```

Response headers:
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 45
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1719307980
```

### Downstream error (502)

```json
{
    "type": "https://api.acme.com/errors/downstream-error",
    "title": "Downstream Service Unavailable",
    "status": 502,
    "detail": "The SAP system is temporarily unavailable. Please retry in a few minutes.",
    "errorCode": "DOWNSTREAM_ERROR",
    "correlationId": "ff0011ee-22dd-33cc-44bb-55aa66778899",
    "timestamp": "2025-06-25T10:33:00Z"
}
```

Note: **Never expose internal system details** (stack traces, database errors, SAP error codes) in 5xx responses to consumers.

### DataWeave error handler (MuleSoft)

```xml
<on-error-continue type="EXPRESSION" logException="false">
    <ee:transform>
        <ee:message>
            <ee:set-payload><![CDATA[
            %dw 2.0
            output application/json
            ---
            {
                type:          "https://api.acme.com/errors/validation-error",
                title:         "Validation Error",
                status:        400,
                detail:        error.description,
                errorCode:     "VALIDATION_ERROR",
                correlationId: vars.correlationId default correlationId,
                timestamp:     now() as String {format: "yyyy-MM-dd'T'HH:mm:ssZ"}
            }
            ]]></ee:set-payload>
        </ee:message>
        <ee:variables>
            <ee:set-variable variableName="httpStatus"><![CDATA[400]]></ee:set-variable>
        </ee:variables>
    </ee:transform>
</on-error-continue>
```

---

## Best Practices

- **Always return a body on errors** — never return an empty 4xx or 5xx response
- **Include a correlationId in every error** — essential for log tracing
- **Use specific status codes** — `422` for business rule failures, not generic `400`
- **Never expose stack traces or internal errors** to consumers
- **Include retry guidance for 429 and 503** — `Retry-After` header
- **Use consistent error structure** across all APIs in your organisation
- **Log errors with the correlationId** — so you can find them in New Relic/Splunk
- **Document your error codes** in your API spec (RAML/OpenAPI)

---

## Common Mistakes

- ❌ Returning `200 OK` with `{ "success": false }` in the body — use correct status codes
- ❌ Using `500` for validation errors — that's a consumer error, use `400` or `422`
- ❌ Exposing exception messages or stack traces in error responses
- ❌ Inconsistent error structures across APIs — consumers can't write generic error handlers
- ❌ No `correlationId` in errors — debugging in production becomes a nightmare
- ❌ Vague error messages: `"Something went wrong"` — be specific about what failed and why

---

## References

- [RFC 7807 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc7807)
- [HTTP Status Codes — IANA Registry](https://www.iana.org/assignments/http-status-codes)
- [REST API Error Handling — Microsoft](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design#error-handling)

---

## Exercises

1. **Status code matching:** Match each scenario to the correct HTTP status code:
   - User submits a form with a missing required field
   - User requests a report they don't have permission to view
   - User deletes a resource successfully (no content to return)
   - API's database connection failed
   - User submits valid JSON but amount is negative (business rule)

2. **Error payload design:** Write the JSON error payload for a payment API that rejects a transfer because the sender's account has insufficient funds.

3. **Anti-pattern fix:** The following error response is badly designed. List every problem and rewrite it:
   ```json
   { "result": "fail", "msg": "NullPointerException at line 42 in PaymentService.java" }
   ```

4. **Error taxonomy:** Define 5 `errorCode` values for a supplier onboarding API. For each, specify the HTTP status code and a sample `detail` message.

5. **MuleSoft implementation:** Write a MuleSoft `on-error-continue` block that catches `HTTP:TIMEOUT` errors and returns a structured 504 error payload with correlationId.

---

## Further Reading

- [Zalando REST API Guidelines — Error Handling](https://opensource.zalando.com/restful-api-guidelines/#errors)
- [Google API Design Guide — Errors](https://cloud.google.com/apis/design/errors)
