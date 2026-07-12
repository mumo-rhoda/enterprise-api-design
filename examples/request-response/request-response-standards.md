# Request/Response Standards

## Standard Request Headers

```http
POST /v1/payments HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440000
Idempotency-Key: 7b4e9120-f38a-42bc-8b11-aa3344556677
User-Agent: PaymentsApp/2.1.0
```

## Standard Response Headers

```http
HTTP/1.1 201 Created
Content-Type: application/json
X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440000
X-Request-ID: server-generated-id
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1719307980
Cache-Control: no-store
```

## Request Body Standards

```json
{
    "comment": "Fields at root level — no wrapping object",
    "amount": 5000.00,
    "currency": "KES",
    "recipient": {
        "accountId": "ACC-001",
        "name": "Acme Ltd"
    },
    "reference": "INV-2025-001",
    "metadata": {
        "source": "mobile-app",
        "version": "2.1.0"
    }
}
```

## Response Body Standards

```json
{
    "comment": "Single resource response — no wrapping",
    "id": "PAY-001",
    "status": "COMPLETED",
    "amount": 5000.00,
    "currency": "KES",
    "recipient": {
        "accountId": "ACC-001",
        "name": "Acme Ltd"
    },
    "createdAt": "2025-06-25T10:30:00Z",
    "updatedAt": "2025-06-25T10:30:05Z"
}
```

```json
{
    "comment": "Collection response — data array + pagination",
    "data": [
        { "id": "PAY-001", "amount": 5000, "status": "COMPLETED" },
        { "id": "PAY-002", "amount": 2000, "status": "PENDING" }
    ],
    "pagination": {
        "total": 250,
        "count": 2,
        "limit": 20,
        "offset": 0,
        "hasMore": true
    }
}
```

## Content Type Rules

| Scenario | Content-Type |
|---|---|
| JSON request/response | `application/json` |
| Problem details (RFC 7807) | `application/problem+json` |
| Form submission | `application/x-www-form-urlencoded` |
| File upload | `multipart/form-data` |
| Plain text | `text/plain` |

## Date/Time Format

Always use **ISO 8601** with UTC:
```
2025-06-25T10:30:00Z          ← datetime (UTC)
2025-06-25T13:30:00+03:00     ← datetime (EAT)
2025-06-25                    ← date only
```

Never use:
```
25/06/2025      ← ambiguous
1719307800      ← epoch (not human-readable)
Jun 25, 2025    ← locale-specific
```
