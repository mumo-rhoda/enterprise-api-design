# Pagination Flow — Sequence Diagram

```
Consumer                    API                        Database
    │                        │                              │
    │  GET /invoices         │                              │
    │  ?limit=20&offset=0   │                              │
    │───────────────────────►│                              │
    │                        │  SELECT ... LIMIT 20        │
    │                        │  OFFSET 0 ORDER BY id DESC  │
    │                        │─────────────────────────────►│
    │                        │◄─────────────────────────────│
    │                        │  COUNT(*)                    │
    │                        │─────────────────────────────►│
    │                        │◄─────────────────────────────│
    │                        │  Build pagination envelope   │
    │  200 + data[20] +      │                              │
    │  pagination.total=240  │                              │
    │◄───────────────────────│                              │
    │                        │                              │
    │  GET /invoices         │                              │
    │  ?limit=20&offset=20  │                              │
    │───────────────────────►│                              │
    │  ... (repeat)          │                              │
```

---

# Cursor Pagination — State Machine

```
START
  │
  ▼
GET /resources?limit=20
  │
  ├── response.pagination.hasMore = true
  │     │
  │     ▼
  │   GET /resources?cursor={nextCursor}&limit=20
  │     │
  │     └── (loop back to hasMore check)
  │
  └── response.pagination.hasMore = false
        │
        ▼
      DONE — all pages consumed
```
