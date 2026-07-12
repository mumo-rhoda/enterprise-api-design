# OpenAPI 3.0

## Introduction

OpenAPI Specification (OAS) is the industry standard for describing REST APIs. An OpenAPI document is a machine-readable contract for your API — it drives documentation, code generation, testing, mocking, and governance tooling.

This section covers OpenAPI 3.0 document structure, authoring best practices, tooling, and integration with enterprise API governance.

---

## Why it matters

- **Single source of truth:** The spec IS the contract — documentation and implementation must match
- **Tooling ecosystem:** Swagger UI, Postman import, code generators, linters, mock servers
- **Governance:** Spec linting enforces naming conventions and standards automatically in CI/CD
- **Onboarding:** New engineers read the spec to understand any API without asking anyone

---

## Prerequisites

- YAML basics
- REST API fundamentals
- HTTP status codes and headers

---

## Architecture

### OpenAPI 3.0 document structure

```yaml
openapi: 3.0.3
info:          ← API metadata (title, version, contact, license)
servers:       ← Base URLs per environment
paths:         ← Endpoints and operations
components:    ← Reusable schemas, responses, parameters, security schemes
security:      ← Global security requirements
tags:          ← Grouping for documentation
```

---

## Implementation

### Full OpenAPI 3.0 example — Customer API

```yaml
openapi: 3.0.3

info:
  title: Customer API
  description: |
    Manages customer records for the enterprise platform.
    Part of the System Layer in the API-led connectivity architecture.
  version: 1.0.0
  contact:
    name: Integration Team
    email: integrations@example.com
  license:
    name: MIT

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://api-staging.example.com/v1
    description: Staging

tags:
  - name: Customers
    description: Customer resource operations

paths:
  /customers:
    get:
      summary: List customers
      description: Returns a paginated list of all customers.
      operationId: listCustomers
      tags: [Customers]
      parameters:
        - $ref: '#/components/parameters/LimitParam'
        - $ref: '#/components/parameters/OffsetParam'
        - $ref: '#/components/parameters/CorrelationIdHeader'
      responses:
        '200':
          description: Paginated list of customers
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerListResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '500':
          $ref: '#/components/responses/InternalError'
      security:
        - bearerAuth: []

    post:
      summary: Create a customer
      operationId: createCustomer
      tags: [Customers]
      parameters:
        - $ref: '#/components/parameters/CorrelationIdHeader'
        - $ref: '#/components/parameters/IdempotencyKeyHeader'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateCustomerRequest'
      responses:
        '201':
          description: Customer created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Customer'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /customers/{customerId}:
    get:
      summary: Get a customer by ID
      operationId: getCustomer
      tags: [Customers]
      parameters:
        - name: customerId
          in: path
          required: true
          schema:
            type: string
            example: C-001
      responses:
        '200':
          description: Customer found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Customer'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    Customer:
      type: object
      required: [id, name, email, createdAt]
      properties:
        id:
          type: string
          description: Unique customer identifier
          example: C-001
        name:
          type: string
          example: Acme Ltd
        email:
          type: string
          format: email
          example: contact@acme.co.ke
        phone:
          type: string
          example: "+254700000000"
        country:
          type: string
          description: ISO 3166-1 alpha-2 country code
          example: KE
        createdAt:
          type: string
          format: date-time
          example: "2025-06-25T10:00:00Z"

    CreateCustomerRequest:
      type: object
      required: [name, email]
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 200
        email:
          type: string
          format: email
        phone:
          type: string
        country:
          type: string
          default: KE

    ErrorResponse:
      type: object
      required: [type, title, status, detail, errorCode, correlationId, timestamp]
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        errorCode:
          type: string
        correlationId:
          type: string
        timestamp:
          type: string
          format: date-time

    CustomerListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Customer'
        pagination:
          type: object
          properties:
            total:   { type: integer }
            count:   { type: integer }
            limit:   { type: integer }
            offset:  { type: integer }
            hasMore: { type: boolean }

  parameters:
    LimitParam:
      name: limit
      in: query
      description: Number of records per page (max 100)
      schema:
        type: integer
        default: 20
        minimum: 1
        maximum: 100

    OffsetParam:
      name: offset
      in: query
      schema:
        type: integer
        default: 0
        minimum: 0

    CorrelationIdHeader:
      name: X-Correlation-ID
      in: header
      description: Unique identifier for distributed tracing
      required: false
      schema:
        type: string
        format: uuid

    IdempotencyKeyHeader:
      name: Idempotency-Key
      in: header
      required: true
      schema:
        type: string
        format: uuid

  responses:
    BadRequest:
      description: Bad request — invalid input
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Conflict:
      description: Resource already exists
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    InternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

---

## Best Practices

- **Use `$ref`** for all reusable schemas, responses, and parameters — no copy-paste
- **Write `operationId`** on every operation — enables clean code generation
- **Include examples** on every schema — makes Swagger UI and mocking useful
- **Document every response code** — including 4xx and 5xx
- **Use `components/responses`** for common error responses — define once, reuse everywhere
- **Lint your spec** in CI/CD — use Spectral or Redocly to enforce standards
- **Version the spec file** alongside your code — treat it as code

---

## Common Mistakes

- ❌ Defining schemas inline instead of using `$ref` — cannot reuse or maintain
- ❌ Missing `operationId` — code generators produce ugly method names
- ❌ Only documenting happy-path responses — consumers have no contract for errors
- ❌ No examples — Swagger UI shows empty schemas, mocking is useless
- ❌ Spec in a separate repo from the code — they drift apart and become lies

---

## References

- [OpenAPI 3.0 Specification](https://spec.openapis.org/oas/v3.0.3)
- [Swagger Editor](https://editor.swagger.io)
- [Spectral — OpenAPI Linter](https://stoplight.io/open-source/spectral)
- [Redocly — OpenAPI Tooling](https://redocly.com)

---

## Exercises

1. **Read a real spec:** Open the [Stripe OpenAPI spec](https://github.com/stripe/openapi) and identify: how they handle pagination, how they structure error schemas, how they define reusable components.

2. **Write a spec:** Write an OpenAPI 3.0 spec for a mobile money transfer API with endpoints: initiate transfer, get transfer status, list transfers.

3. **Linting:** Install Spectral (`npm install -g @stoplight/spectral-cli`) and lint the OpenAPI example in this repo. Fix any violations.

4. **Schema design:** Design the OpenAPI schema for an M-Pesa STK Push payment request including all required and optional fields.

---

## Further Reading

- [OpenAPI Best Practices — Swagger Blog](https://swagger.io/blog/api-design/openapi-tips/)
- [API Design Patterns — JJ Geewax](https://www.manning.com/books/api-design-patterns)
