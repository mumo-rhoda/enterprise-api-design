# API Naming — Good vs Bad Examples

## Resource URLs

| Bad ❌ | Good ✅ | Violation |
|---|---|---|
| `GET /getCustomer` | `GET /customers/{id}` | Verb in URL |
| `GET /Customer` | `GET /customers` | Singular noun |
| `GET /customer_orders` | `GET /customer-orders` | Underscore instead of hyphen |
| `POST /createPayment` | `POST /payments` | Verb in URL |
| `DELETE /removeUser/123` | `DELETE /users/123` | Verb in URL |
| `GET /api/v1/GetInvoiceById?id=INV-001` | `GET /v1/invoices/INV-001` | Query param for ID, verb in URL |
| `GET /customers.json` | `GET /customers` (Accept: application/json) | File extension |

## Field Naming

| Bad ❌ | Good ✅ | Convention |
|---|---|---|
| `CustomerID` | `customerId` | camelCase for JSON fields |
| `customer_id` | `customerId` | No underscores in JSON |
| `CUST_ID` | `customerId` | No ALL_CAPS |
| `dt_created` | `createdAt` | Meaningful names, not abbreviations |
| `amt` | `amount` | No abbreviations |
| `flg_active` | `isActive` | Boolean prefix: `is`, `has`, `can` |

## Header Naming

| Bad ❌ | Good ✅ |
|---|---|
| `x_correlation_id` | `X-Correlation-ID` |
| `correlationid` | `X-Correlation-ID` |
| `idempotencykey` | `Idempotency-Key` |
| `Authorization_Token` | `Authorization` |

## Error Code Naming

| Bad ❌ | Good ✅ |
|---|---|
| `err1` | `VALIDATION_ERROR` |
| `error_invalid` | `INVALID_FIELD_VALUE` |
| `500` | `INTERNAL_SERVER_ERROR` |
| `oops` | `DOWNSTREAM_TIMEOUT_ERROR` |
