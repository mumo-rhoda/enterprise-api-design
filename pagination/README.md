# Pagination

## Introduction

Pagination controls how large datasets are broken into manageable chunks for API consumers. Without pagination, a single request for all customers could return millions of records — crashing the client, exhausting server memory, and creating unacceptable latency.

This section covers offset pagination, cursor pagination, keyset pagination, response envelope design, and HATEOAS navigation links.

---

## Why it matters

- **Performance:** Returning 10,000 records per request is slow for everyone
- **Predictability:** Consumers need to know how much data to expect per page
- **Safety:** Prevents runaway queries from overwhelming databases or APIs
- **UX:** Frontend apps need paginated data to render lists efficiently

---

## Prerequisites

- Understanding of REST resource design
- Basic SQL knowledge (for keyset/cursor concepts)

---

## Architecture

### Pagination strategies compared

```
OFFSET PAGINATION
  GET /customers?offset=0&limit=20   → records 1-20
  GET /customers?offset=20&limit=20  → records 21-40
  ✅ Simple to implement
  ❌ Unstable: inserts/deletes shift pages
  ❌ Slow on large datasets (DB must scan all preceding rows)
  ❌ Cannot paginate reliably in real-time data

CURSOR PAGINATION
  GET /customers?cursor=eyJpZCI6MTAwfQ&limit=20
  ✅ Stable: cursor points to a position, not an offset
  ✅ Consistent even with concurrent writes
  ❌ Cannot jump to arbitrary page
  ❌ Cursor must be opaque to consumers

KEYSET PAGINATION
  GET /customers?after_id=100&limit=20
  ✅ Efficient: DB uses index on sort key
  ✅ Stable on inserts
  ❌ Sort key must be unique and indexed
  ❌ Cannot sort on arbitrary fields efficiently
```

---

## Implementation

### Standard response envelope

```json
{
    "data": [...],
    "pagination": {
        "total":    1500,
        "count":    20,
        "limit":    20,
        "offset":   40,
        "hasMore":  true
    },
    "_links": {
        "self":     "/v1/customers?offset=40&limit=20",
        "first":    "/v1/customers?offset=0&limit=20",
        "prev":     "/v1/customers?offset=20&limit=20",
        "next":     "/v1/customers?offset=60&limit=20",
        "last":     "/v1/customers?offset=1480&limit=20"
    }
}
```

### Offset pagination request

```http
GET /v1/customers?offset=0&limit=20&sort=createdAt&order=desc HTTP/1.1
Accept: application/json
X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440000
```

### Cursor pagination response

```json
{
    "data": [...],
    "pagination": {
        "count":      20,
        "limit":      20,
        "hasMore":    true,
        "nextCursor": "eyJpZCI6MTIwLCJjcmVhdGVkQXQiOiIyMDI1LTA2LTI1In0="
    },
    "_links": {
        "next": "/v1/customers?cursor=eyJpZCI6MTIwLCJjcmVhdGVkQXQiOiIyMDI1LTA2LTI1In0=&limit=20"
    }
}
```

### DataWeave — building a paginated response

```dataweave
%dw 2.0
output application/json

var pageSize  = vars.limit  as Number default 20
var pageStart = vars.offset as Number default 0
var total     = vars.totalCount as Number
var hasMore   = (pageStart + pageSize) < total

---
{
    data: payload,
    pagination: {
        total:   total,
        count:   sizeOf(payload),
        limit:   pageSize,
        offset:  pageStart,
        hasMore: hasMore
    },
    "_links": {
        self:  "/v1/customers?offset=$(pageStart)&limit=$(pageSize)",
        first: "/v1/customers?offset=0&limit=$(pageSize)",
        next:  if (hasMore) "/v1/customers?offset=$(pageStart + pageSize)&limit=$(pageSize)" else null,
        prev:  if (pageStart > 0) "/v1/customers?offset=$(pageStart - pageSize)&limit=$(pageSize)" else null
    }
}
```

### Recommended defaults and limits

```
Default page size:  20
Maximum page size:  100
Enforce maximum:    return 400 if limit > 100
Sort default:       createdAt DESC
```

---

## Best Practices

- **Always paginate collection endpoints** — no unbounded list responses
- **Enforce a maximum page size** — prevent consumers requesting 10,000 records
- **Include `hasMore`** — consumers know when to stop paginating
- **Include HATEOAS `_links`** — consumers don't need to construct pagination URLs
- **Use cursor pagination** for real-time or high-churn datasets
- **Use offset pagination** for simple, admin, or report-style APIs
- **Return `total` count** — helps consumers show "Page 3 of 75" in UI
- **Document defaults** in your RAML/OpenAPI spec

---

## Common Mistakes

- ❌ No pagination — returning all records in one response
- ❌ Allowing unlimited `limit` values — consumers will abuse this
- ❌ Using `page` + `pageSize` inconsistently — pick one convention and stick to it
- ❌ Offset pagination on high-churn data — records appear or disappear between pages
- ❌ Not including navigation links — consumers must hardcode URL construction
- ❌ Not including `total` in the response — consumers can't build progress indicators

---

## References

- [Stripe Pagination Design](https://stripe.com/docs/api/pagination)
- [GitHub REST API Pagination](https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api)
- [Zalando REST Guidelines — Pagination](https://opensource.zalando.com/restful-api-guidelines/#pagination)

---

## Exercises

1. **Strategy selection:** Which pagination strategy would you choose for each of these, and why?
   - A report API returning historical transaction data (stable, no real-time updates)
   - A social feed API where new posts appear every second
   - An admin API listing all system users (< 10,000 records total)

2. **Response design:** Design the full JSON response envelope for a paginated endpoint returning invoices (page 3 of 12, 20 per page, 240 total).

3. **Edge cases:** What should your API return when:
   - `offset` exceeds `total`?
   - `limit` is 0?
   - `limit` is 10,000 (above your maximum)?

4. **DataWeave exercise:** Write a DataWeave transformation that takes a flat array of records and wraps it in a pagination envelope, given `limit`, `offset`, and `total` as variables.

---

## Further Reading

- [Evolving API Pagination at Slack](https://slack.engineering/evolving-api-pagination-at-slack/)
- [Cursor-based Pagination — Max Countryman](https://engineering.fb.com/2008/08/25/core-data/efficient-pagination-using-mysql/)
