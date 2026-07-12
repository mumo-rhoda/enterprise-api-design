# RAML 1.0 for MuleSoft

## Introduction

RAML (RESTful API Modeling Language) is MuleSoft's API specification format. It is YAML-based, human-readable, and tightly integrated with the Anypoint Platform — Design Center, Exchange, and API Manager all consume RAML natively.

This section covers RAML 1.0 document structure, data types, resource types, traits, fragments, and Exchange publishing.

---

## Why it matters

- **Anypoint-native:** RAML drives Design Center, mocking, documentation, and contract enforcement in API Manager
- **Reusability:** RAML fragments (data types, traits, resource types) can be shared across API specs via Exchange
- **Design-first:** Writing the RAML before the implementation forces API design decisions early
- **Governance:** RAML specs published to Exchange become the governed contract

---

## Prerequisites

- YAML syntax
- REST API fundamentals
- Anypoint Platform access (free tier works)

---

## Architecture

### RAML document structure

```
api.raml                 ← Root spec
├── types/               ← Reusable data types
│   ├── Customer.raml
│   ├── Error.raml
│   └── Pagination.raml
├── traits/              ← Reusable behaviours (pagination, error-handling)
│   ├── pageable.raml
│   └── secured.raml
├── resource-types/      ← Reusable resource patterns
│   └── collection.raml
└── examples/            ← Example request/response payloads
    ├── customer.json
    └── customer-list.json
```

### Exchange fragment types

```
RAML Fragments
├── Data Type        ← Reusable schema (shared across APIs)
├── Trait            ← Reusable behaviour (pagination, auth headers)
├── Resource Type    ← Reusable resource pattern (CRUD collection)
├── Security Scheme  ← Reusable auth (OAuth 2.0, JWT, Client ID)
└── Example          ← Reusable example payload
```

---

## Implementation

### Full RAML 1.0 example — Customer API

```yaml
#%RAML 1.0
title: Customer API
description: |
  System Layer API for managing customer records.
  Part of the API-led connectivity architecture.
version: v1
baseUri: https://api.example.com/{version}
mediaType: application/json
protocols: [HTTPS]

uses:
  types: exchange_modules/customer-types/1.0.0/customer-types.raml

traits:
  pageable:
    queryParameters:
      limit:
        type: integer
        default: 20
        maximum: 100
        minimum: 1
        description: Records per page
      offset:
        type: integer
        default: 0
        minimum: 0
        description: Starting record index
  secured:
    headers:
      Authorization:
        type: string
        description: Bearer token
        example: Bearer eyJhbGciOiJSUzI1NiJ9...
  correlatable:
    headers:
      X-Correlation-ID:
        type: string
        description: Unique request trace ID (UUID v4)
        example: 550e8400-e29b-41d4-a716-446655440000

/customers:
  description: Customer collection
  get:
    is: [pageable, secured, correlatable]
    description: Returns a paginated list of customers
    responses:
      200:
        body:
          application/json:
            type: CustomerListResponse
            example: !include examples/customer-list.json
      400:
        body:
          application/json:
            type: ErrorResponse
            example: !include examples/error-400.json
      401:
        body:
          application/json:
            type: ErrorResponse

  post:
    is: [secured, correlatable]
    description: Creates a new customer
    headers:
      Idempotency-Key:
        type: string
        description: UUID v4 for idempotent creation
        required: true
    body:
      application/json:
        type: CreateCustomerRequest
        example: !include examples/create-customer.json
    responses:
      201:
        body:
          application/json:
            type: Customer
            example: !include examples/customer.json
      400:
        body:
          application/json:
            type: ErrorResponse
      409:
        body:
          application/json:
            type: ErrorResponse

  /{customerId}:
    description: Single customer resource
    uriParameters:
      customerId:
        type: string
        description: Unique customer identifier
        example: C-001

    get:
      is: [secured, correlatable]
      description: Retrieves a single customer by ID
      responses:
        200:
          body:
            application/json:
              type: Customer
        404:
          body:
            application/json:
              type: ErrorResponse

    patch:
      is: [secured, correlatable]
      description: Partially updates a customer record
      body:
        application/json:
          type: UpdateCustomerRequest
      responses:
        200:
          body:
            application/json:
              type: Customer
        404:
          body:
            application/json:
              type: ErrorResponse

    delete:
      is: [secured, correlatable]
      description: Deletes a customer record
      responses:
        204:
          description: Customer deleted successfully
        404:
          body:
            application/json:
              type: ErrorResponse
```

### RAML Data Types

```yaml
#%RAML 1.0 DataType
# types/Customer.raml
displayName: Customer
type: object
properties:
  id:
    type: string
    description: Unique customer identifier
    example: C-001
    required: true
  name:
    type: string
    minLength: 1
    maxLength: 200
    required: true
  email:
    type: string
    pattern: "^[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}$"
    required: true
  phone:
    type: string
    required: false
  country:
    type: string
    description: ISO 3166-1 alpha-2
    default: KE
    required: false
  createdAt:
    type: datetime
    required: true
```

### RAML Trait — Pageable

```yaml
#%RAML 1.0 Trait
# traits/pageable.raml
usage: Apply to any collection GET endpoint
queryParameters:
  limit:
    displayName: Limit
    type: integer
    default: 20
    maximum: 100
    description: Maximum number of records to return
  offset:
    displayName: Offset
    type: integer
    default: 0
    description: Starting position in the collection
responses:
  400:
    description: Invalid pagination parameters
```

---

## Best Practices

- **Design-first:** Write the RAML spec before writing any Mule flow code
- **Use fragments:** Share data types and traits via Exchange across API specs
- **Use `!include`** for examples — keep JSON examples in separate files
- **Apply traits** to every endpoint that needs pagination, security, or correlation
- **Publish to Exchange** — every API spec should be discoverable
- **Validate** your RAML with the Anypoint Design Center parser
- **Version your RAML** alongside your Mule application in the same Git repo

---

## Common Mistakes

- ❌ Defining data types inline instead of using named types
- ❌ Not using traits — copying pagination parameters to every endpoint
- ❌ Not publishing to Exchange — the spec is never discovered or reused
- ❌ Keeping RAML in a separate repo from the Mule project — they drift
- ❌ No examples — mocking service and documentation are useless without them

---

## References

- [RAML 1.0 Specification](https://raml.org/developers/raml-100-tutorial)
- [MuleSoft Design Center Documentation](https://docs.mulesoft.com/design-center/)
- [Anypoint Exchange — Publishing APIs](https://docs.mulesoft.com/exchange/to-publish-assets-maven)

---

## Exercises

1. **Write a RAML spec:** Design a RAML 1.0 spec for a payment initiation API. Include: POST to initiate, GET to check status, correct traits for all endpoints.

2. **Extract a trait:** Take the RAML example above and extract the error response pattern into a reusable `error-handling` trait that can be applied with `is: [error-handling]`.

3. **Data type design:** Write a RAML data type for an M-Pesa STK Push request. Include validation constraints (required fields, min/max lengths, patterns).

4. **Fragment planning:** You're building 5 APIs for a fintech platform. Identify which RAML fragments (data types, traits, resource types) should be shared across all 5 and why.

---

## Further Reading

- [MuleSoft RAML Best Practices](https://blogs.mulesoft.com/api-integration/strategy/raml-best-practices/)
- [RAML vs OpenAPI — When to use each](https://docs.mulesoft.com/general/api-led-connectivity)
