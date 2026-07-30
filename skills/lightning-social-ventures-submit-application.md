---
name: Submit an application to a Lightning Reach support scheme
description: >-
  Discover which support schemes the authenticated organisation can apply to,
  submit an application on a client's behalf, and read back its state.
api: openapi/lightning-social-ventures-lightning-reach-openapi.json
base_url: https://api.lightningreach.org
operations:
  - PublicGrantsController_findAll_v1
  - PublicGrantApplicationsController_submitApplication_v1
  - PublicGrantApplicationsController_findById_v1
---

# Submit an application to a Lightning Reach support scheme

Lightning Reach connects people in financial hardship to grants, vouchers and
support schemes run by councils, housing associations, charities and utilities.
This skill covers the submission path a partner system uses to lodge an
application on a client's behalf.

## Before you start

- **Authentication is required but undocumented.** Every endpoint returns
  `401 {"statusCode":401,"message":"UnauthorizedException"}` without a valid
  partner credential, and the OpenAPI declares no `securitySchemes`. Use the
  credential and header your Lightning Reach onboarding contact supplied; do not
  guess a scheme. Credentials are scoped to an **organisation**, not a user.
- **This API handles personal financial-hardship data** — names, emails,
  household circumstances. Never log request or response bodies, never echo
  applicant data into a transcript, and only act on a client's behalf when you
  hold their consent.
- All requests and responses are `application/json`. Responses are `no-store`.

## Step 1 — find the schemes you can apply to

Call `PublicGrantsController_findAll_v1` (`GET /v1/support-schemes`). It returns
the schemes available to the authenticated organisation.

Each scheme carries the vocabulary you will need later:

- `id` — the value you pass as `supportId` when applying
- `frozen` / `frozenCopy` — a frozen scheme is not accepting changes
- `stages` — the scheme's own workflow stages
- `awardReasons`, `reviewReasons`, `declineReasons` — the per-scheme reason lists

Pick the scheme by `id`. Do not hard-code a scheme id across environments.

## Step 2 — submit the application

Call `PublicGrantApplicationsController_submitApplication_v1`
(`POST /v1/applications`) with a `CreateApplicationDto`:

Required fields:

- `firstName` (string)
- `lastName` (string)
- `supportId` (string) — the scheme `id` from step 1
- `declarationOfTruth` (boolean) — must reflect a genuine declaration made by
  the client. Never set this to `true` on your own initiative.
- `modules` (array of `ModuleDto`) — each with a `moduleId` and a `data` object
  keyed as a two-level config path, e.g.
  `{"moduleId": "...", "data": {"info-sharing": {"consent": true, "informationSharing": true}}}`

Optional fields:

- `email` (string, `format: email`)
- `skipSubmissionEmails` (boolean) — suppresses the applicant's confirmation
  email; only use it when your own system sends the notification instead.

A success returns `201`.

> **There is no idempotency contract.** The API exposes no `Idempotency-Key`
> header or parameter. If a POST times out, **do not blind-retry** — call
> `PublicGrantApplicationsController_findAll_v1` with the `search` parameter
> (it matches client name, email or application reference) to check whether the
> application already landed, then retry only if it did not.

## Step 3 — read the application back

Call `PublicGrantApplicationsController_findById_v1`
(`GET /v1/applications/{id}`) to confirm state. Pass
`includeDataDictionary=true` when you need the task data dictionary alongside
the submitted values.

The response (`PublicApplicationDataResponseDto`) includes `status`, `stage`,
`applicationTasks`, `clients`, `reference` and a numeric `version`.

`status` is one of: `INVITED`, `STARTED`, `SUBMITTED`, `AWAITING_HOUSEHOLD`,
`INFO_REQUESTED`, `REVIEWED`, `DECLINED`, `APPROVED`, `OPTED_OUT`, `INACTIVE`,
`INELIGIBLE`.

A freshly submitted application should read `SUBMITTED`. If it reads
`AWAITING_HOUSEHOLD` or `INFO_REQUESTED`, more input is needed from the client
or their household before it can progress — surface that to a human rather than
attempting to fill it in.

## Error handling

Errors use the envelope `{"statusCode": <int>, "message": "<string>"}` — this is
**not** RFC 9457 `application/problem+json`. The spec documents no 4xx or 5xx
responses, so treat any non-2xx as opaque: log the `statusCode`, surface the
`message`, and stop. Do not infer retry semantics that are not documented.

Responses carry a W3C `traceparent` header — capture it when reporting a problem
to Lightning Reach support (`hello@lightningreach.org`).
