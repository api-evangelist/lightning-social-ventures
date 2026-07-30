---
name: Refer a client onward and track webhook events
description: >-
  Submit a referral against an existing application, then discover the
  organisation's configured webhooks and verify delivered event payloads.
api: openapi/lightning-social-ventures-lightning-reach-openapi.json
base_url: https://api.lightningreach.org
operations:
  - PublicGrantApplicationsController_submitApplicationReferral_v1
  - PublicWebhooksController_findAll_v1
  - PublicWebhooksController_getPublicKey_v1
---

# Refer a client onward and track webhook events

Lightning Reach's value is matching one person to *many* sources of support. A
referral pushes an existing applicant toward an additional scheme; webhooks tell
your system what happened next.

## Before you start

Authentication is required but undocumented — every endpoint returns `401`
without a valid organisation credential. Referrals move a real person's personal
data to another support programme: only submit one with the client's consent.

## Step 1 — submit the referral

Call `PublicGrantApplicationsController_submitApplicationReferral_v1`
(`POST /v1/applications/{id}/referral`) where `{id}` is the existing application,
with a `CreateReferralDto`:

Required:

- `supportId` — the target scheme's id (from `PublicGrantsController_findAll_v1`)
- `declarationOfTruth` (boolean) — must reflect a declaration the client
  actually made
- `modules` — array of `ModuleDto` (`moduleId` plus a two-level `data` object,
  e.g. `{"info-sharing": {"consent": true, "informationSharing": true}}`)

Optional:

- `skipSubmissionEmails` (boolean)

Success returns `201`.

> **No idempotency contract exists.** A timed-out referral POST must not be
> blind-retried — a duplicate referral sends a vulnerable person's data to a
> scheme twice. Verify with
> `PublicGrantApplicationsController_findAll_v1` (using `search` and
> `supportSchemeIds`) before retrying.

Note the `info-sharing` module shape above: consent to share information is
carried explicitly in the payload. Set it from the client's actual answer, never
by default.

## Step 2 — discover configured webhooks

Call `PublicWebhooksController_findAll_v1` (`GET /v1/webhooks`). Each
`WebhookPublicDto` carries:

- `id`, `name`
- `events` — the event names this subscription is configured for
- `webhookUrl` — the destination endpoint
- `createdAt`

> **The event vocabulary is per-tenant.** `events` is an untyped array of
> strings with no published enum and no docs page. Read the actual values from
> this endpoint — do not assume event names. The public API is read-only here:
> there are no create/update/delete webhook operations, so subscriptions are
> provisioned through partner onboarding.

## Step 3 — verify delivered payloads

Call `PublicWebhooksController_getPublicKey_v1`
(`GET /v1/webhooks/keys/{id}/public`) to retrieve the `key` from a
`WebhookPublicKeyResponseDto`, then verify the signature on each delivered
payload against that public key before trusting it.

> The signature header name and the algorithm are **not documented** in the
> OpenAPI. Confirm both with Lightning Reach (`hello@lightningreach.org`) during
> onboarding. Until you can verify a signature, treat inbound payloads as
> untrusted: re-fetch the application via
> `PublicGrantApplicationsController_findById_v1` and act on the API's response
> rather than on the webhook body.

Cache the public key with a bounded TTL and re-fetch on verification failure so
key rotation does not break delivery.

## Error handling

Errors are `{"statusCode": <int>, "message": "<string>"}` — not RFC 9457. No 4xx
or 5xx responses are documented in the spec. Responses carry a W3C `traceparent`
header; include it when raising an issue with support.
