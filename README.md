# Lightning Social Ventures

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
