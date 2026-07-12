# HTTP Status Code Reference — Enterprise API

## Quick Decision Guide

```
Was it successful?
  YES →
    Did you create something new? → 201 Created
    Is there content to return?  → 200 OK
    No content to return?        → 204 No Content
    Async, processing later?     → 202 Accepted

  NO →
    Is it the consumer's fault (4xx)?
      Missing/invalid auth?       → 401 Unauthorized
      Authed but no permission?   → 403 Forbidden
      Resource doesn't exist?     → 404 Not Found
      Malformed input?            → 400 Bad Request
      Valid input, business rule? → 422 Unprocessable Entity
      Rate limit hit?             → 429 Too Many Requests
      Duplicate resource?         → 409 Conflict

    Is it the server's fault (5xx)?
      Unexpected error?           → 500 Internal Server Error
      Downstream system failed?   → 502 Bad Gateway
      Planned maintenance?        → 503 Service Unavailable
      Downstream timed out?       → 504 Gateway Timeout
```

## Common Misuses to Avoid

| Wrong ❌ | Correct ✅ | Why |
|---|---|---|
| `200` with `{ "success": false }` | `400` or `422` | 200 means success |
| `500` for validation errors | `400` | Validation is consumer's fault |
| `404` for auth failures | `401` or `403` | Don't reveal resource existence to unauthorised callers |
| `200` for created resource | `201` | 201 signals creation specifically |
| `400` for business rule violation | `422` | 422 = valid format, failed logic |
| `200` for no-content DELETE | `204` | 204 explicitly signals no body |
