# Correlation ID Propagation

## What is a Correlation ID?

A Correlation ID is a unique identifier generated at the entry point of a request that is propagated through every service, log line, and downstream call. It ties together all activity for a single logical request — essential for distributed tracing.

## Generation

The first service to receive the request generates the ID if not already present:

```dataweave
%dw 2.0
output application/json
---
vars.correlationId default uuid()
```

Or in MuleSoft, the built-in `correlationId` variable:
```
#[correlationId]   // Mule's built-in correlation ID per message
```

## Propagation — MuleSoft example

```xml
<!-- On entry: extract or generate -->
<set-variable variableName="correlationId"
              value="#[attributes.headers['X-Correlation-ID'] default correlationId]"
              doc:name="Set Correlation ID"/>

<!-- Log with correlation ID on every logger -->
<logger level="INFO"
        message="#['[' ++ vars.correlationId ++ '] Processing request: ' ++ attributes.requestPath]"/>

<!-- Pass downstream -->
<http:request method="GET" url="...">
    <http:headers>
        <http:header key="X-Correlation-ID" value="#[vars.correlationId]"/>
    </http:headers>
</http:request>

<!-- Return in response -->
<http:response statusCode="#[vars.httpStatus default 200]">
    <http:headers>
        <http:header key="X-Correlation-ID" value="#[vars.correlationId]"/>
    </http:headers>
</http:response>
```

## Log format with Correlation ID

```json
{
    "timestamp": "2025-06-25T10:30:00Z",
    "level": "ERROR",
    "correlationId": "550e8400-e29b-41d4-a716-446655440000",
    "appName": "payments-notification-api",
    "flow": "process-notification-flow",
    "message": "Downstream SAP connection timed out after 5000ms"
}
```

## New Relic NRQL — trace by correlationId

```sql
SELECT message, level, flow, timestamp
FROM Log
WHERE correlationId = '550e8400-e29b-41d4-a716-446655440000'
ORDER BY timestamp ASC
```

## Standard Header Name

Always use: `X-Correlation-ID`  
Format: UUID v4  
Example: `550e8400-e29b-41d4-a716-446655440000`
