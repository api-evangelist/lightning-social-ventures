---
name: Review an application and record an award decision
description: >-
  Work an application queue, inspect the submitted evidence and files, then
  record an approve or decline decision with the awards granted.
api: openapi/lightning-social-ventures-lightning-reach-openapi.json
base_url: https://api.lightningreach.org
operations:
  - PublicGrantApplicationsController_findAll_v1
  - PublicGrantApplicationsController_findById_v1
  - PublicGrantApplicationsController_getFiles_v1
  - PublicGrantApplicationsController_updateStatusById_v1
  - PublicGrantsController_findAll_v1
---

# Review an application and record an award decision

This is the grant-assessor path: pull the queue, read the evidence, and write a
decision back. **The final step disburses or denies financial support to a
person in hardship — it is a high-consequence write and must not be automated
without a human decision-maker.**

## Before you start

- Authentication is required but undocumented; every endpoint returns `401`
  without a valid organisation credential. See
  `authentication/lightning-social-ventures-authentication.yml`.
- Never log applicant data. Treat every field as sensitive personal data.

## Step 1 — pull the queue

Call `PublicGrantApplicationsController_findAll_v1` (`GET /v1/applications`).

Filters:

- `supportSchemeIds` — restrict to one or more schemes
- `status` — e.g. `SUBMITTED` or `REVIEWED` to find work awaiting a decision
- `search` — client name, email or application reference
- `latestSubmissionFrom` / `latestSubmissionTo` — ISO 8601 timestamp bounds

Paging and sorting:

- `page`, `pageSize` — page-number pagination; the response carries `count`
  (total matching) alongside `grantApplications` (the page)
- `sortKey` — one of `clientName`, `supportScheme`, `reference`, `createdAt`,
  `firstSubmissionAt`, `latestSubmissionAt`
- `sortOrder` — `ASC` or `DESC`

Page until you have consumed `count` items; defaults and the maximum `pageSize`
are not documented, so read `count` rather than assuming a page size.

## Step 2 — read the full application

Call `PublicGrantApplicationsController_findById_v1`
(`GET /v1/applications/{id}`) with `includeDataDictionary=true` so the submitted
`data` in each `applicationTasks` entry can be interpreted against its
dictionary.

Note the `version` number — it identifies the revision you assessed.

## Step 3 — inspect the uploaded evidence

Call `PublicGrantApplicationsController_getFiles_v1`
(`GET /v1/applications/{id}/assets`), optionally with `includeFormData=true`.

Each `PublicFileResponseDto` carries:

- `fileName`, `taskId`, `version`
- `downloadUrl` — **expires in 30 minutes**; fetch it when you are ready to
  read the file, do not cache or persist it
- `scanStatus` — `PENDING`, `SCANNED` or `QUARANTINED`

> **Only open files with `scanStatus: SCANNED`.** `PENDING` means the malware
> scan has not finished — wait and re-fetch. `QUARANTINED` means the scan
> failed: do not download it, and flag it to a human.

## Step 4 — record the decision

Call `PublicGrantApplicationsController_updateStatusById_v1`
(`PUT /v1/applications/{id}/status`) with an `UpdateApplicationStatusDto`.

Required:

- `status` — one of `INVITED`, `STARTED`, `SUBMITTED`, `AWAITING_HOUSEHOLD`,
  `REVIEWED`, `DECLINED`, `APPROVED`, `OPTED_OUT`, `INACTIVE`, `INELIGIBLE`.
  (Note `INFO_REQUESTED` is readable on an application but is **not** in the
  set this operation accepts.)

Optional:

- `stage` — use a stage from that scheme's `stages` list (step 0 / step 1)
- `decisionReason` — use a value from the scheme's `awardReasons`,
  `reviewReasons` or `declineReasons` as appropriate to the status
- `decisionDetails`, `note`
- `awardDate`
- `awards` — array of `AwardDto`

Each `AwardDto` requires a `type`: an award code from the LSV taxonomy, e.g.
`LSV0500` (Energy and utilities), `LSV0700` (Food and essential items),
`LSV0400` (Devices and digital access), `LSV1300` (Education and training -
adult), `LSV3000` (Benefit entitlement check). The full 29-code list is in
`data-model/lightning-social-ventures-data-model.yml`.

Optional per award: `value` (number, minimum 1), `paymentType` (one of
`BACS: applicant payment`, `BACS: supplier payment`,
`Purchase from preferred Supplier`, `Vouchers`, `Cheques`, `Gift Card`,
`Electronic debit/credit card`), `paymentDate`, `details`.

Codes ending in "please specify" — `LSV0100`, `LSV2000`, `LSV5000` — require a
`details` string to be meaningful.

The response is an `UpdateStatusResponseDto` with the updated application `id`.

> **Guardrails.** Fetch the scheme's own `awardReasons` / `declineReasons` via
> `PublicGrantsController_findAll_v1` and pick from them rather than inventing a
> reason string. Re-read the application (step 2) and confirm `version` has not
> moved before writing. There is no idempotency key on this endpoint — if the
> call times out, GET the application and check its `status` before retrying.

## Error handling

Errors are `{"statusCode": <int>, "message": "<string>"}`, not RFC 9457. No 4xx
or 5xx responses are documented. Never retry a status write blindly; verify
first.
