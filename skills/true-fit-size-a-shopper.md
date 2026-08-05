---
name: Size a shopper for a retailer product
description: Establish a True Fit user and profile for one of your users, load what you know about their body and wardrobe, then return a specific recommended size for up to 100 retailer products.
api: openapi/true-fit-partner-api-openapi.json
operations: [syncIds, listProfiles, createProfile, upsertMeasurements, createClosetItem, getBulkRecommendation, getBulkGeneralGuidance]
generated: '2026-08-05'
method: generated
source: https://techdocs.truefitcorp.com/reference/partner-overview
---

# Size a shopper for a retailer product

Base URL `https://partner.truefitcorp.com/api`. Authenticate with HTTP Basic where the
**username is empty** and the password is the partner API key —
`Authorization: Basic base64(":" + apiKey)`. The leading colon is required.

## Steps

1. **Sync the user once.** Call `syncIds` (`GET /id-sync`) with your `partnerId` and your
   own `partnerUserId`. This endpoint is signed with HMAC-SHA256, not Basic auth. Persist
   the returned `tfPartnerUserId` and drive every later request from it.
2. **List profiles.** Call `listProfiles` (`GET /partner/{partnerId}/profile`) with
   `tfPartnerUserId`. A user may hold several profiles, one per person they shop for;
   there is no implicit "current" profile — always address one explicitly by `profileId`.
3. **Create a profile if the list is empty.** Call `createProfile`
   (`POST /partner/{partnerId}/profile`) with the `department` (`womens`, `mens`, `girls`,
   `boys`, `unisexkids`, `maternity`, …). Enum values match case-insensitively but are
   stored lowercase.
4. **Give the profile something to size against.** A profile with neither measurements nor
   closet items returns `incompleteProfile`.
   - `upsertMeasurements` (`PUT /partner/{partnerId}/profile/{profileId}/measurements`)
     replaces the whole measurement set. Use `MeasurementPoint` names (`height`,
     `naturalWaist`, `inseam`, `footLength`, …) with a `Uom` (`cm`, `in`, `kg`, `lb`, …),
     or the general scale `verySmall` / `small` / `typical` / `big` / `veryBig`.
   - `createClosetItem` (`POST /partner/{partnerId}/profile/{profileId}/closet`) records a
     garment they already own. The `brand`, `department`, `category`, `sellingLocale`,
     `hierarchyLabels` and `sellingSize` must match True Fit's catalogue — a mismatch
     returns 500 and retrying the identical payload will not succeed.
5. **Wait out body estimation.** Profile, measurement and closet writes trigger asynchronous
   body estimation. A recommendation requested immediately after one may return
   `userEstimationInProgress`; retry after a short delay.
6. **Ask for sizes.** Call `getBulkRecommendation`
   (`POST /partner/{partnerId}/profile/{profileId}/recommendation/bulk`) with a JSON array
   of 1–100 `{retailerDomain, productId, locale}` items. The response is an array of the
   **same length in the same order** — match by position. Present `recommendedSize` for
   every item with `success: true`.
7. **Fall back for the rest.** For items with `success: false` — or when the shopper has no
   profile at all — call `getBulkGeneralGuidance`
   (`POST /partner/{partnerId}/general-guidance/bulk`). It needs no user identifier and
   returns `recommendationSummary` ("Runs small") and `recommendationMessage`.

## Rules

- `retailerDomain` accepts a hostname or a full URL, but only the hostname is used and
  `www.` is **not** stripped. Confirm with True Fit which exact hostnames are registered.
- One unrecognised product does not fail the batch; the HTTP status is still a success
  status and the failure is on that item's `error` field. See
  `errors/true-fit-error-codes.yml` for the full outcome vocabulary.
- A `404` on your first call means an unknown or disabled `partnerId` — it is checked
  **before** credentials — not a bad key. See `errors/true-fit-problem-types.yml`.
- There is no idempotency key and no pagination. See `conventions/true-fit-conventions.yml`.
- Do not send PII. True Fit does not collect or accept names, email addresses or postal
  addresses on any integration point.
