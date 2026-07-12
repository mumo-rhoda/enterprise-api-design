# Idempotency

## Introduction

An operation is **idempotent** if making the same request multiple times produces the same result as making it once. Idempotency is essential for building safe, retryable integrations — especially over unreliable networks or with at-least-once message delivery systems.

---

## Why it matters

Networks fail. Timeouts happen. Consumers retry. Without idempotency:
- A payment retried after a timeout could be processed twice
- A supplier could be created three times because of a retry loop
- A message from Anypoint MQ (at-least-once delivery) processes a record multiple times

With idempotency, retries are safe — the second, third, and tenth request produce exactly the same outcome as the first.

---

## Prerequisites

- HTTP method semantics (GET, POST, PUT, DELETE)
- Basic understanding of distributed systems and network failures
- Familiarity with MuleSoft Object Store v2 or a cache

---

## Architecture

### HTTP methods and idempotency

```
NATURALLY IDEMPOTENT
  GET    ← Reading the same record N times = same result
  PUT    ← Replacing with the same payload N times = same state
  DELETE ← Deleting the same record N times = same state (gone)
  HEAD   ← Like GET, just metadata

NOT NATURALLY IDEMPOTENT
  POST   ← Creating a record N times = N records ❌
  PATCH  ← Incrementing a field N times ≠ same state ❌
```

### Making POST idempotent with Idempotency-Key

```
Client → POST /v1/payments
         Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

API  → Check Object Store: has this key been processed?
         NO  → Process payment → Store result under key → Return 201
         YES → Return stored result → Return 200 (or 201, be consistent)
```

### Idempotency lifecycle

```
Request arrives with Idempotency-Key
       │
       ▼
Key in cache?
  ├── NO  → Process request → Store response in cache (TTL: 24h) → Return 201
  └── YES → Return cached response → Return 200/201
```

---

## Implementation

### Idempotency-Key header standard

```http
POST /v1/payments HTTP/1.1
Content-Type: application/json
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
X-Correlation-ID: 7b4e9120-f38a-42bc-8b11-aa3344556677

{
    "amount": 5000,
    "currency": "KES",
    "recipient": "ACC-001"
}
```

Rules:
- Client generates a UUID v4 as the key
- Key must be unique per logical operation (not per retry)
- Key is bound to the request payload — same key + different payload = 422 Conflict
- Keys expire after a configurable TTL (typically 24 hours)

### MuleSoft implementation using Object Store v2

```xml
<!-- Step 1: Extract idempotency key -->
<set-variable variableName="idempotencyKey"
              value="#[attributes.headers['Idempotency-Key'] default null]"
              doc:name="Extract Idempotency Key"/>

<!-- Step 2: Check if already processed -->
<os:retrieve key="#[vars.idempotencyKey]"
             objectStore="IdempotencyStore"
             target="cachedResponse"
             doc:name="Check Idempotency Cache"/>
```

```dataweave
%dw 2.0
output application/json
---
// If key exists in cache → return cached response
// If not → process and cache
if (vars.cachedResponse != null)
    vars.cachedResponse
else
    // process payment...
    payload
```

```xml
<!-- Step 3: Store response after processing -->
<os:store key="#[vars.idempotencyKey]"
          objectStore="IdempotencyStore"
          doc:name="Cache Response">
    <os:value>#[payload]</os:value>
</os:store>
```

### Object Store v2 configuration

```xml
<os:object-store name="IdempotencyStore"
                 persistent="true"
                 maxEntries="10000"
                 entryTtl="86400"
                 entryTtlUnit="SECONDS"
                 doc:name="Idempotency Object Store"/>
```

### Error: Conflicting request (same key, different payload)

```json
{
    "type":       "https://api.acme.com/errors/idempotency-conflict",
    "title":      "Idempotency Key Conflict",
    "status":     422,
    "detail":     "The Idempotency-Key '550e8400...' was previously used with a different request payload.",
    "errorCode":  "IDEMPOTENCY_CONFLICT",
    "correlationId": "...",
    "timestamp":  "2025-06-25T10:30:00Z"
}
```

---

## Examples

### Full request-response cycle

**First request (processes):**
```http
POST /v1/payments HTTP/1.1
Idempotency-Key: abc123

{ "amount": 5000, "currency": "KES" }

→ HTTP 201 Created
{ "paymentId": "PAY-001", "status": "PROCESSED" }
```

**Retry (returns cached):**
```http
POST /v1/payments HTTP/1.1
Idempotency-Key: abc123

{ "amount": 5000, "currency": "KES" }

→ HTTP 201 Created   ← same status code as original
{ "paymentId": "PAY-001", "status": "PROCESSED" }   ← same body
```

Note: Return the **original status code**, not `200`. If original was `201`, return `201` again.

---

## Best Practices

- **Require Idempotency-Key on all non-idempotent endpoints** (POST, PATCH)
- **Validate the key is a UUID** — reject malformed keys with 400
- **Store the full response**, not just a flag — return identical responses on retry
- **Use 24-hour TTL** as a starting point; adjust based on business needs
- **Bind key to payload hash** — detect and reject conflicting requests
- **Return the original status code on replay** — not just 200
- **Log idempotency cache hits** — useful for detecting retry storms

---

## Common Mistakes

- ❌ Not implementing idempotency on payment or order creation endpoints
- ❌ Returning `200` on replay when original was `201` — confuses consumers
- ❌ Not binding the key to the payload — allows reuse of keys with different data
- ❌ Too short a TTL — retries after the window get double-processed
- ❌ Using a sequential ID as the idempotency key — must be consumer-generated UUID

---

## References

- [Stripe Idempotency](https://stripe.com/docs/api/idempotent_requests)
- [AWS API Gateway — Idempotency](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [MuleSoft Object Store v2](https://docs.mulesoft.com/object-store/)

---

## Exercises

1. **Classify operations:** For each operation, state whether it is naturally idempotent and why:
   - `GET /invoices/INV-001`
   - `POST /payments`
   - `PUT /customers/C001`
   - `DELETE /sessions/S001`
   - `PATCH /accounts/A001` (incrementing balance by 100)

2. **Implementation design:** Design the idempotency flow for a supplier creation API. What do you store in the cache? What TTL would you use? What happens if the same key is sent with different supplier data?

3. **MQ idempotency:** You're consuming messages from Anypoint MQ (at-least-once delivery). A message is delivered twice with the same `messageId`. Write the DataWeave logic to detect and skip the duplicate.

4. **Conflict scenario:** A mobile app times out waiting for a payment response. It retries with the same `Idempotency-Key` but a different `amount`. What should your API return and why?

---

## Further Reading

- [Idempotency in Distributed Systems — Exactly Once Processing](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
- [Designing Idempotent APIs — Thoughtworks](https://www.thoughtworks.com/insights/blog/idempotency-api)
