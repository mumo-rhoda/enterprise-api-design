# API-Led Connectivity — Architecture Diagram

```
                        ┌─────────────────────────────────────────────────────┐
                        │                  CONSUMERS                          │
                        │  Mobile App │  Web App  │  Partner System │  ERP   │
                        └──────┬──────────┬───────────────┬─────────────┬────┘
                               │          │               │             │
                        ┌──────▼──────────▼───────────────▼─────────────▼────┐
                        │               EXPERIENCE LAYER                      │
                        │  mobile-banking-api │ partner-api │ internal-api    │
                        │  (tailored per consumer channel)                    │
                        └──────────────────────────┬──────────────────────────┘
                                                   │  orchestrates
                        ┌──────────────────────────▼──────────────────────────┐
                        │                PROCESS LAYER                         │
                        │  payment-processing-api │ customer-onboarding-api   │
                        │  (business logic, rules, orchestration)              │
                        └──────────┬──────────────────────────┬───────────────┘
                                   │  accesses                │
                   ┌───────────────▼───────────┐  ┌──────────▼────────────────┐
                   │       SYSTEM LAYER         │  │       SYSTEM LAYER        │
                   │  sap-system-api            │  │  salesforce-system-api    │
                   │  (thin wrapper, no logic)  │  │  coupa-system-api         │
                   └───────────────┬────────────┘  └──────────┬────────────────┘
                                   │                          │
                   ┌───────────────▼────────────┐  ┌──────────▼────────────────┐
                   │    SAP ERP                 │  │    Salesforce CRM         │
                   │    (PI/PO, MDG, BTP)       │  │    Coupa Procurement      │
                   └────────────────────────────┘  └───────────────────────────┘
```

---

# Request Lifecycle — Sequence Diagram

```
Mobile App          Experience API       Process API         System API          SAP
    │                     │                   │                   │               │
    │ POST /payments/send │                   │                   │               │
    │────────────────────►│                   │                   │               │
    │                     │ POST /process-pmt │                   │               │
    │                     │──────────────────►│                   │               │
    │                     │                   │ GET /accounts/bal  │               │
    │                     │                   │──────────────────►│               │
    │                     │                   │                   │ IDoc/RFC call │
    │                     │                   │                   │──────────────►│
    │                     │                   │                   │◄──────────────│
    │                     │                   │◄──────────────────│               │
    │                     │                   │ POST /payments/exe │               │
    │                     │                   │──────────────────►│               │
    │                     │                   │◄──────────────────│               │
    │                     │◄──────────────────│                   │               │
    │◄────────────────────│                   │                   │               │
    │ 201 Created         │                   │                   │               │
```

---

# Error Propagation — Sequence Diagram

```
Consumer           API Gateway          Mule App            Downstream
    │                   │                   │                   │
    │  POST /payments   │                   │                   │
    │──────────────────►│                   │                   │
    │                   │  POST /payments   │                   │
    │                   │──────────────────►│                   │
    │                   │                   │  POST /system     │
    │                   │                   │──────────────────►│
    │                   │                   │      TIMEOUT      │
    │                   │                   │◄── 504 ───────────│
    │                   │                   │                   │
    │                   │  {error payload}  │                   │
    │                   │◄──────────────────│                   │
    │ 502 + error JSON  │                   │                   │
    │◄──────────────────│                   │                   │
```
