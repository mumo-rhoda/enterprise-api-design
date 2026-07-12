# API Versioning

## Introduction

API versioning is the strategy for managing changes to an API over time without breaking existing consumers. A versioning strategy defines how version information is communicated and how breaking vs non-breaking changes are handled.

---

## Why it matters

APIs are contracts between providers and consumers. When you change an API without a versioning strategy:
- Consumer applications break silently or loudly
- You lose the ability to evolve independently
- Rollback becomes impossible
- Trust erodes — internal or external consumers stop relying on your APIs

A versioning strategy gives you freedom to evolve while protecting consumers.

---

## Prerequisites

- Understanding of REST API design basics
- Familiarity with HTTP headers
- Basic knowledge of semantic versioning (SemVer)

---

## Architecture

### When is a change "breaking"?

```
NON-BREAKING (safe — no version bump needed)
├── Adding a new optional field to a response
├── Adding a new optional query parameter
├── Adding a new endpoint
└── Returning more data than before (if consumers ignore unknown fields)

BREAKING (requires new version)
├── Removing a field from a response
├── Renaming a field
├── Changing a field's data type
├── Changing an endpoint URL
├── Removing an endpoint
├── Making an optional field required
└── Changing authentication method
```

### Version lifecycle

```
v1 (current) ──────────────────────────────────► active
v2 (new)     ───────────────────────────────────► active
v1           ──────────── DEPRECATED ────────────► sunset date announced
v1                                               ► retired (removed)
```

---

## Implementation

### Strategy 1: URI Versioning (recommended for most enterprise APIs)

```
https://api.example.com/v1/customers
https://api.example.com/v2/customers
```

**Pros:** Visible in logs, easy to test, easy to route at gateway level  
**Cons:** "Unclean" URLs (purists argue versions aren't a resource property)

**MuleSoft / Anypoint implementation:**
```yaml
# RAML
baseUri: https://{domain}/v1
version: v1
```

### Strategy 2: Header Versioning

```http
GET /customers HTTP/1.1
API-Version: 2
```

**Pros:** Clean URLs  
**Cons:** Not visible in browser, harder to test, caching complications

### Strategy 3: Accept Header (Media Type) Versioning

```http
GET /customers HTTP/1.1
Accept: application/vnd.mycompany.v2+json
```

**Pros:** Follows HTTP spec strictly  
**Cons:** Complex, rarely used outside large public APIs (GitHub uses this)

### Recommended Enterprise Standard: URI Versioning

```
/v{major}
```

Only bump the major version on breaking changes:
- `v1` → `v2` on breaking change
- Non-breaking changes go into the same version

### Deprecation headers

When a version is deprecated, include in every response:

```http
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
Link: <https://api.example.com/v2/customers>; rel="successor-version"
```

---

## Examples

### URI versioning in MuleSoft RAML

```yaml
#%RAML 1.0
title: Customer API
version: v1
baseUri: https://api.example.com/{version}
mediaType: application/json

/customers:
  get:
    description: List all customers
    responses:
      200:
        body:
          application/json:
            example: !include examples/customers-list.json
```

### Version routing at API Gateway

```
Client → API Gateway → Route by path prefix:
  /v1/* → Customer API v1 (CloudHub app: customer-api-v1)
  /v2/* → Customer API v2 (CloudHub app: customer-api-v2)
```

### Deprecation notice in OpenAPI 3.0

```yaml
paths:
  /v1/customers:
    get:
      summary: List customers (DEPRECATED — use /v2/customers)
      deprecated: true
      description: |
        **Deprecated.** This endpoint will be retired on 2025-12-31.
        Please migrate to /v2/customers.
```

---

## Best Practices

- **Default to URI versioning** for enterprise and internal APIs
- **Only version on breaking changes** — non-breaking changes go in the same version
- **Announce deprecations at least 6 months in advance** for internal APIs, 12 months for external
- **Never remove a version without a sunset date** — give consumers time to migrate
- **Support maximum 2 versions simultaneously** — 3+ creates unsustainable maintenance
- **Keep changelogs** — document every change per version
- **Automate version validation** in CI/CD — fail pipeline if breaking change detected without version bump

---

## Common Mistakes

- ❌ Using `/v1.1`, `/v1.2` — use major versions only, minor changes are non-breaking
- ❌ Never deprecating — eventually you have 7 active versions
- ❌ Breaking change without version bump — consumers wake up to broken production
- ❌ Including version in the resource name: `/customers-v2` — use path prefix
- ❌ Versioning every change — only breaking changes need a new version
- ❌ No sunset communication — consumers can't plan migration

---

## References

- [Stripe API Versioning Strategy](https://stripe.com/blog/api-versioning)
- [RFC 8594 — Sunset Header](https://www.rfc-editor.org/rfc/rfc8594)
- [Deprecation Header — IETF Draft](https://tools.ietf.org/html/draft-dalal-deprecation-header)
- [MuleSoft RAML Versioning](https://docs.mulesoft.com/api-manager/latest/api-versioning)

---

## Exercises

1. **Breaking vs non-breaking:** For each change below, classify as breaking or non-breaking:
   - Adding `middleName` (optional) to a customer response
   - Renaming `phone` to `phoneNumber`
   - Removing `legacyId` from the response
   - Adding a new `GET /customers/{id}/preferences` endpoint
   - Changing `amount` from a string to a number

2. **Deprecation notice:** Write the HTTP response headers for a v1 endpoint being sunset on March 31, 2026, with v2 as the successor.

3. **Versioning strategy:** Your team is building a public payments API for a Kenyan fintech. Define your versioning strategy: which approach, how you'll handle deprecation, and how long you'll maintain old versions.

4. **Changelog exercise:** Write a changelog entry for a v2 release that renames `customerId` to `id`, adds optional `tags` field, and removes the deprecated `legacyCode` field.

---

## Further Reading

- *The Design of Web APIs* — Arnaud Lauret (Chapter 8: Versioning)
- [Semantic Versioning](https://semver.org)
- [API Change Management — Postman Blog](https://blog.postman.com/api-versioning-best-practices/)
