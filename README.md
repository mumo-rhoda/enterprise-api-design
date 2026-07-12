# Enterprise API Design

> A comprehensive reference guide for designing, governing, and documenting enterprise-grade APIs.

Built from production experience integrating SAP, Coupa, Salesforce, and cloud platforms using MuleSoft.  
Shared by the **MuleSoft Meetup Nairobi** community and **Castriq** integration consultancy.

---

## 📚 Contents

| Section | Topics covered |
|---|---|
| [`/api-standards`](api-standards/) | API-led connectivity, resource naming, HTTP methods, REST maturity |
| [`/versioning`](versioning/) | URI versioning, header versioning, deprecation strategy |
| [`/error-handling`](error-handling/) | Error payload structure, HTTP status codes, error taxonomies |
| [`/pagination`](pagination/) | Cursor, offset, keyset pagination; response envelopes |
| [`/idempotency`](idempotency/) | Idempotency keys, at-least-once delivery, safe retries |
| [`/openapi`](openapi/) | OpenAPI 3.0 spec authoring, tooling, governance |
| [`/raml`](raml/) | RAML 1.0 for MuleSoft, fragments, Exchange publishing |
| [`/examples`](examples/) | Working code and spec examples for every topic |
| [`/diagrams`](diagrams/) | Architecture and sequence diagrams |
| [`/images`](images/) | Supporting visuals and reference images |

---

## 🗂️ Documentation standard

Every section in this repo follows the same structure:

1. **Introduction** — what this topic is
2. **Why it matters** — the business and technical case
3. **Prerequisites** — what you should know first
4. **Architecture** — how it fits into the bigger picture
5. **Implementation** — step-by-step guidance
6. **Examples** — working code and spec samples
7. **Best Practices** — what good looks like
8. **Common Mistakes** — what to avoid
9. **References** — official sources
10. **Exercises** — hands-on practice
11. **Further Reading** — go deeper

---

## 🏗️ Topics covered

- **API-led Connectivity** — 3-layer architecture (Experience / Process / System)
- **Resource Naming** — nouns, plurals, hierarchy, path conventions
- **HTTP Status Codes** — correct usage, common misuses
- **REST Maturity** — Richardson Maturity Model (levels 0–3)
- **Versioning** — URI vs header vs media type versioning
- **Correlation IDs** — distributed tracing, propagation patterns
- **Naming Conventions** — fields, endpoints, headers, errors
- **Enterprise API Governance** — lifecycle, contracts, breaking changes
- **OpenAPI** — spec authoring, validation, code generation
- **RAML** — MuleSoft API spec, fragments, data types, Exchange
- **Request/Response Standards** — envelopes, content types, headers
- **Error Payloads** — structured errors, problem+json, RFC 7807

---

## 🚀 Who this is for

- Integration engineers building enterprise APIs on MuleSoft, Spring Boot, or any platform
- API architects defining governance standards for their organisation
- Engineers preparing for the MuleSoft MCIA-I exam
- Teams onboarding to API-led connectivity

---

## 🤝 Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). PRs welcome — especially additional examples, diagrams, and real-world case studies from East African enterprise contexts.

---

## 👩🏾‍💻 Author

**Rhoda Mumo** — Senior Integration Engineer · MuleSoft MCIA Candidate  
Founder, MuleSoft Meetup Nairobi · Co-founder, Citova  
[LinkedIn](https://www.linkedin.com/in/rhodamumo) · [Portfolio](https://rhodamumo.vercel.app) · Nairobi, Kenya

---

## 📄 License

MIT — see [`LICENSE`](LICENSE)
