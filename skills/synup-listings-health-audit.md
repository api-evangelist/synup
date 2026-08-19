---
name: synup-listings-health-audit
description: Audit a Synup location's directory presence — premium, voice and AI listings, duplicate detection and merge, indexing rate, and Google verification state.
api: Synup API v4
base_url: https://api.synup.com/api/v4
generated: '2026-08-13'
method: generated
source: openapi/synup-api-openapi.yml
operations:
  - getPremiumListings
  - getVoiceListings
  - GET /locations/{locationId}/ai-listings
  - getDuplicateListings
  - getAccountDuplicateListings
  - markAsDuplicate
  - markAsNotDuplicate
  - GET /locations/{locationId}/indexing-rate
  - googleVerificationLocationStats
---

# Audit listings health for a location

The point of this flow is a coverage report that means something: not "we are on 80
directories" but which listings are premium, which are voice-enabled, which are duplicates
fragmenting the review count, and how fast the network is actually indexing changes.

## Steps

1. **Premium coverage.** `getPremiumListings` — `GET /locations/{locationId}/listings/premium`.
2. **Voice coverage.** `getVoiceListings` — `GET /locations/{locationId}/voice-assistants`
   returns the Alexa/Siri/Assistant surfaces for the location.
3. **AI listings.** `GET /locations/{locationId}/ai-listings` returns listings tuned for how
   LLM and AI search surfaces parse the profile.
4. **Find duplicates.** `getDuplicateListings` for one location, or
   `getAccountDuplicateListings` (`GET /locations/listings/duplicates`) to sweep the account.
5. **Resolve them.** `markAsDuplicate` (`POST /locations/listings/mark-as-duplicate`) or
   `markAsNotDuplicate`. Both are destructive-ish judgments — record what you merged.
6. **Measure propagation.** `GET /locations/{locationId}/indexing-rate` reports how much of
   the location's data has actually landed downstream.
7. **Check Google verification.** `googleVerificationLocationStats`
   (`GET /locations/google-verification-stats`) rolls verification state up across locations.

## Errors that matter here

`SY00002` listing item id not found · `SY00003` invalid site ids · `SY00004` location not yet
approved · `SY00008` location is archived · `SY00009` no access to location. A location in
`archived` state answers most listing reads with `SY00008`; activate it with
`activateLocations` before auditing.

## Event-driven variant

Rather than polling, subscribe to the webhook feed: `listing.submission` (sync success or
incomplete), `connection.listing_inaccessible`, `connection.reauth_required`,
`connection.google_verification_verified` and `connection.google_verification_failed`. One
account-wide URL receives all of them; there is no per-event subscription.
