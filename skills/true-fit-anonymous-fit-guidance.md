---
name: Answer "does this run small?" with no shopper profile
description: Get aggregate, profile-free fit guidance for up to 100 retailer products so an agent can answer how a garment runs before any registration or identification has happened.
api: openapi/true-fit-partner-api-openapi.json
operations: [getBulkGeneralGuidance]
generated: '2026-08-05'
method: generated
source: https://techdocs.truefitcorp.com/reference/getbulkgeneralguidance
---

# Answer "does this run small?" with no shopper profile

`getBulkGeneralGuidance` (`POST /partner/{partnerId}/general-guidance/bulk`) is the only
Partner API operation that requires **no user identifier**. It is the correct call for an
anonymous visitor, a product page before registration, and any agent turn that has no
consented shopper identity — and the documented fallback when `getBulkRecommendation`
returns `incompleteProfile` or `noProfile`.

## Steps

1. Authenticate with HTTP Basic — empty username, partner API key as the password.
2. POST a JSON array of 1–100 items, each `{retailerDomain, productId, locale}`. `locale`
   is `language_COUNTRY` (e.g. `en_US`) and defaults to the retailer's primary locale.
3. Read the response array positionally — same length, same order, each item echoing the
   `retailerDomain`, `productId` and `locale` you sent.
4. For `success: true`, surface `recommendationSummary` as the headline and
   `recommendationMessage` as the explanation. Both are localized to `locale`.
5. For `success: false`, map the `error` string:
   - `Invalid retailerDomain` — the value could not be parsed as a hostname.
   - `Unsupported retailer domain` — the hostname is not mapped to a retailer.
   - `No general guidance returned for product` — the product is unknown, or too few
     shoppers have rated it to produce guidance. Say nothing rather than guessing.

## Rules

- Never synthesise fit advice from the product description or reviews when this call
  returns no guidance — the point of the call is that the answer is grounded in real
  purchase-and-return outcomes.
- Guidance is aggregate. Do not present it as personalised; escalate to
  `getBulkRecommendation` once the shopper has a profile with measurements or closet items.
