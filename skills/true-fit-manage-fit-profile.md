---
name: Manage a shopper's fit profile, measurements and closet
description: Create, read, update and delete the True Fit profiles a partner user shops for, together with the body measurements and owned-garment closet items that make those profiles sizable.
api: openapi/true-fit-partner-api-openapi.json
operations: [syncIds, listProfiles, createProfile, updateProfile, deleteProfile, getMeasurements, upsertMeasurements, listClosetItems, createClosetItem, updateClosetItem, deleteClosetItem]
generated: '2026-08-05'
method: generated
source: https://techdocs.truefitcorp.com/reference/partner-overview
---

# Manage a shopper's fit profile, measurements and closet

## Identify first

Every operation below except `syncIds` needs a user identifier as a query parameter —
`tfPartnerUserId` (preferred, returned by `syncIds`) or `partnerUserId` (your own id).
Omitting both returns `400`. Sending both makes `tfPartnerUserId` win. An identifier that
was never synced returns `401 Unknown partner user`, which means *call `syncIds` first*,
not *bad credentials*.

## Profiles

- `listProfiles` — `GET /partner/{partnerId}/profile`. Returns every person this user shops
  for. Ignore `isActive`: it is always `false` for partner requests, which carry no browser
  session.
- `createProfile` — `POST /partner/{partnerId}/profile`. Profiles you create record your
  `partnerId` as their `originStore`.
- `updateProfile` — `PUT /partner/{partnerId}/profile/{profileId}`.
- `deleteProfile` — `DELETE /partner/{partnerId}/profile/{profileId}`. A `404` means the
  `profileId` does not exist or does not belong to the identified user.

## Measurements

- `getMeasurements` — `GET /partner/{partnerId}/profile/{profileId}/measurements`.
- `upsertMeasurements` — `PUT .../measurements`. This **replaces** the measurement set, so
  send the full set you want stored rather than a delta. Values are either exact
  (`{value, uom}`) or the general scale `verySmall` / `small` / `typical` / `big` /
  `veryBig`.

## Closet

- `listClosetItems` — `GET /partner/{partnerId}/profile/{profileId}/closet`.
- `createClosetItem` — `POST .../closet`.
- `updateClosetItem` — `PUT .../closet/{closetItemId}`.
- `deleteClosetItem` — `DELETE .../closet/{closetItemId}`.

A closet item that does not match a size in True Fit's catalogue returns `500`. That is not
a transient failure — verify `brand`, `department`, `category`, `sellingLocale`,
`hierarchyLabels` and `sellingSize` before retrying.

## Rules

- Every write triggers asynchronous body estimation. Expect `userEstimationInProgress` on a
  recommendation requested immediately afterwards and retry after a short delay.
- Validation failures return `{message, error}` where `error` maps failing field paths to
  messages — and, unlike every other error body, carries **no** `statusCode`.
- Enum values (`Department`, `ClosetCategory`, `SizeType`, `Uom`) match case-insensitively
  and are stored and returned lowercase.
- Do not send PII of any kind.
