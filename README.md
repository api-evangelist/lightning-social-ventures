# Lightning Social Ventures

Lightning Social Ventures is the UK social-impact company behind **Lightning Reach**, a financial
support platform that connects people facing hardship to grants, benefits, vouchers and assistance
schemes through a single profile and eligibility match. Councils, housing associations, charities,
utilities and banks use it to run their own support programmes end to end — intake, evidence review,
decision, award and payment.

Backed by: techstars — https://lightningreach.org/

## API

Lightning Reach operates a partner-facing REST API.

| | |
|---|---|
| Base URL | https://api.lightningreach.org |
| Reference | https://api.lightningreach.org/docs (Swagger UI) |
| Version | v1 (URI path) |
| Operations | 9 across Applications, Support Schemes, Webhooks |
| Auth | Required on every endpoint, but **not documented** — no `securitySchemes` in the spec |

The OpenAPI 3.0.0 document is not served at a stable spec URL; it is embedded in the Swagger UI
bootstrap at `/docs/swagger-ui-init.js`, from which it was harvested.

## Artifacts

| Dir | Artifact | Method |
|---|---|---|
| `openapi/` | Lightning Reach API, OpenAPI 3.0.0, 9 ops / 15 schemas | searched |
| `llms/` | `llms.txt` published at lightningreach.org | searched |
| `mcp/` | Live Wix Site MCP endpoint (verified), plus derived candidate tools | searched |
| `asyncapi/` | Webhook surface + public-key signature verification | derived |
| `skills/` | 3 Agent Skills grounded in real operationIds | generated |
| `overlays/` | Overlay 1.0.0 of API Evangelist enhancements | generated |
| `data-model/` | 8 entities, 12 relationships, the 29-code LSV award taxonomy | derived |
| `conventions/` | Pagination, versioning, tracing, caching, file handling | derived |
| `errors/` | NestJS `{statusCode,message}` envelope + domain outcome vocabularies | derived |
| `lifecycle/` | Versioning; absence of status page / deprecation policy / SLA | derived |
| `conformance/` | Standards asserted and not asserted | derived |
| `authentication/` | Probed auth posture — required, undocumented | probed |
| `agentic-access/` | `x-agentic-access` contracts for all 9 operations | generated |
| `security/` | TLS/HSTS/DNSSEC/SPF/DMARC domain security | probed |
| `well-known/` | Negative probe record — nothing published | probed |

## Notable gaps

- No `securitySchemes` in the OpenAPI, though every endpoint returns `401`.
- No `servers[]` in the OpenAPI.
- No error responses documented — the spec carries only 200/201.
- No idempotency contract on the application- and referral-creating POSTs.
- No status page, deprecation policy, SLA, changelog, or `.well-known` documents.
- No published webhook event catalog, signature header, or algorithm.
- No SDKs, CLI, Postman collection, or GitHub organisation.
