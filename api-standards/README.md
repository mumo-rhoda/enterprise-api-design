# API Standards

## Introduction

API standards are the agreed rules that govern how APIs are designed, named, and behave across an organisation. They ensure consistency, predictability, and reusability — so any developer can pick up any API and immediately understand it.

This section covers the foundational standards every enterprise API must follow: API-led connectivity, resource naming, HTTP method usage, and the REST maturity model.

---

## Why it matters

Without standards:
- Every team designs APIs differently → consumers must learn each one from scratch
- Refactoring one API breaks its consumers unpredictably
- Governance and security policies cannot be applied uniformly
- Reuse is impossible — similar functionality gets rebuilt repeatedly

With standards:
- APIs become self-describing and predictable
- New engineers onboard in hours, not weeks
- API Manager policies apply consistently across all APIs
- Business units can compose APIs into new products without rework

**Real example:** At E.ON, MuleSoft integrates SAP, Coupa, and 20+ enterprise systems.  
Without API-led connectivity standards, each integration would be a point-to-point spaghetti connection.  
With API-led, a single SAP System API serves data to three Process APIs and six Experience APIs — built once, reused everywhere.

---

## Prerequisites

- Basic understanding of HTTP (methods, status codes, headers)
- Familiarity with REST concepts
- Optional: MuleSoft Anypoint Platform account

---

## Architecture

### API-Led Connectivity — 3-Layer Model

```
┌──────────────────────────────────────────────────────────┐
│                   EXPERIENCE LAYER                        │
│  Consumer-specific APIs — tailored per channel            │
│                                                           │
│  mobile-banking-api  │  partner-api  │  internal-ops-api │
└────────────────────────────┬─────────────────────────────┘
                             │  orchestrates
┌────────────────────────────▼─────────────────────────────┐
│                    PROCESS LAYER                          │
│  Business logic, orchestration, transformation            │
│                                                           │
│  payment-processing-api  │  customer-onboarding-api       │
└────────────────────────────┬─────────────────────────────┘
                             │  accesses
┌────────────────────────────▼─────────────────────────────┐
│                    SYSTEM LAYER                           │
│  Thin wrappers around backend systems                     │
│                                                           │
│  sap-system-api  │  salesforce-system-api  │  db-api      │
└──────────────────────────────────────────────────────────┘
```

### Resource Naming Hierarchy

```
/v1
  /customers                          ← collection
    /{customerId}                     ← single resource
      /accounts                       ← sub-collection
        /{accountId}                  ← sub-resource
          /transactions               ← nested collection
            /{transactionId}
```

---

## Implementation

### Resource Naming Rules

| Rule | Wrong ❌ | Right ✅ |
|---|---|---|
| Use nouns, not verbs | `GET /getCustomer` | `GET /customers/{id}` |
| Use plural nouns | `GET /customer` | `GET /customers` |
| Use lowercase | `GET /CustomerOrders` | `GET /customer-orders` |
| Use hyphens, not underscores | `GET /customer_orders` | `GET /customer-orders` |
| No file extensions | `GET /customers.json` | `GET /customers` (use Accept header) |
| No trailing slashes | `GET /customers/` | `GET /customers` |

### HTTP Methods — Correct Usage

| Method | Purpose | Idempotent? | Body? |
|---|---|---|---|
| `GET` | Retrieve resource(s) | ✅ Yes | ❌ No |
| `POST` | Create new resource | ❌ No | ✅ Yes |
| `PUT` | Replace entire resource | ✅ Yes | ✅ Yes |
| `PATCH` | Partial update | ❌ No | ✅ Yes |
| `DELETE` | Remove resource | ✅ Yes | Optional |
| `HEAD` | GET without body (metadata check) | ✅ Yes | ❌ No |
| `OPTIONS` | Discover allowed methods (CORS) | ✅ Yes | ❌ No |

### Richardson Maturity Model

| Level | Name | Description | Example |
|---|---|---|---|
| 0 | The Swamp of POX | One endpoint, one method, RPC-style | `POST /api { "action": "getCustomer" }` |
| 1 | Resources | Separate URIs per resource | `GET /customers/123` |
| 2 | HTTP Verbs | Correct HTTP method per operation | `DELETE /customers/123` |
| 3 | Hypermedia (HATEOAS) | Responses include links to next actions | `{ "_links": { "transactions": "/customers/123/transactions" } }` |

> **Enterprise target:** Level 2 minimum. Level 3 for public or partner APIs.

---

## Examples

See [`/examples/rest-maturity/`](../examples/rest-maturity/) for full before/after examples at each maturity level.

### Level 0 → Level 2 transformation

**Level 0 (avoid):**
```http
POST /api HTTP/1.1
Content-Type: application/json

{ "action": "getCustomer", "id": "C001" }
```

**Level 2 (target):**
```http
GET /v1/customers/C001 HTTP/1.1
Accept: application/json
X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440000
```

---

## Best Practices

- Design APIs for **consumers**, not systems — ask "what does the caller need?"
- Keep System APIs **thin** — no business logic, just system access
- Keep Experience APIs **consumer-specific** — mobile and web may need different fields
- Use Process APIs to **orchestrate** — never call System APIs directly from Experience
- **Publish every API to Exchange** — discoverability enables reuse
- **Apply policies in API Manager** — not in code
- Version from **day one** — retrofitting versioning is painful

---

## Common Mistakes

- ❌ Putting business logic in a System API
- ❌ Calling a System API directly from an Experience API (skipping Process layer)
- ❌ Using verbs in resource names: `/createPayment`, `/deleteUser`
- ❌ Using singular nouns: `/customer` instead of `/customers`
- ❌ Using `POST` for everything (Level 0 trap)
- ❌ Exposing database table names as resource names
- ❌ Designing APIs for one consumer — design for reuse from day one

---

## References

- [MuleSoft API-Led Connectivity Whitepaper](https://www.mulesoft.com/resources/api/apis-great-digital-transformation)
- [Richardson Maturity Model — Martin Fowler](https://martinfowler.com/articles/richardsonMaturityModel.html)
- [REST API Design Best Practices — Google](https://cloud.google.com/apis/design)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)

---

## Exercises

1. **Classify existing APIs:** Take 3 APIs from your organisation. Classify each at a Richardson Maturity level. What would it take to move each to Level 2?

2. **Resource naming audit:** Review 5 API endpoints. For each, identify any naming violations and rewrite the URL correctly.

3. **API-led mapping:** Draw the 3-layer API-led diagram for a customer onboarding flow at a Kenyan bank. Which systems sit at each layer?

4. **HTTP method audit:** For each operation below, identify the correct HTTP method and explain why:
   - Retrieve a list of invoices
   - Update a supplier's email address only
   - Replace an entire purchase order
   - Delete a payment record
   - Check if a resource exists without downloading it

5. **Design a mini API:** Design the resource model (URLs + HTTP methods only) for a mobile money transfer API serving M-Pesa, bank transfer, and paybill payments.

---

## Further Reading

- *RESTful Web APIs* — Leonard Richardson, Mike Amundsen
- *The Design of Web APIs* — Arnaud Lauret
- MuleSoft MCIA Study Guide — Domain 2: API Design
