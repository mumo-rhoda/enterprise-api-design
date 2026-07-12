# Idempotency Flow — Decision Diagram

```
Consumer sends POST /payments
with Idempotency-Key: abc-123
         │
         ▼
  ┌─────────────────────────────────────────┐
  │  Check Object Store for key 'abc-123'   │
  └─────────────────────────────────────────┘
         │
         ├── KEY NOT FOUND ──────────────────────────────────────────────────►
         │                                                                    │
         │                                                          Process payment
         │                                                                    │
         │                                                          Store result in OS
         │                                                          with key 'abc-123'
         │                                                          TTL = 24h
         │                                                                    │
         │                                                          Return 201 Created
         │
         └── KEY FOUND ───────────────────────────────────────────────────────►
                                                                              │
                                                                   Check payload hash
                                                                              │
                                                             ┌────────────────┴────────────────┐
                                                             │                                 │
                                                    Same payload                        Different payload
                                                             │                                 │
                                                   Return cached response           Return 422 Conflict
                                                   (same status as original)        "Idempotency-Key
                                                             │                       conflict"
                                                   Return 201 Created
```
