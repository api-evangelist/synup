---
name: synup-onboard-a-location
description: Create a business location in Synup, put it in the right folder and tags, attach photos, and confirm it is syndicating to the directories on the plan.
api: Synup API v4
base_url: https://api.synup.com/api/v4
generated: '2026-08-13'
method: generated
source: openapi/synup-api-openapi.yml
operations:
  - GET /countries
  - GET /sub-categories
  - GET /plan-sites
  - createLocation
  - createFolder
  - addLocationToTag
  - uploadSingleLocationPhoto
  - checkBulkPhotoUploadStatus
  - listLocations
---

# Onboard a business location

Every other Synup capability hangs off a location, so this is the first flow. Locations
are validated hard on the way in — the error catalog has more than 120 codes for this one
resource — so validate before you POST rather than after.

## Before you call

- Auth header is `Authorization: API <your_api_key>` — the literal string `API `, a space,
  then the key. It is **not** `Bearer`. Send `Content-Type: application/json` on every call.
- There are no idempotency keys. A retried `createLocation` creates a second location, and
  Synup will reject it with `SY10017` (*Location with same name, phone number and zip code
  has already been added*) only if all three fields match. Record the returned id before retrying.
- Check `errors[]` on every response even when the HTTP status is 200 — Synup returns
  `{"data": ..., "errors": [...]}` together and may populate both.

## Steps

1. **Resolve the address vocabulary.** `GET /countries` returns supported countries and
   their states. An unsupported country is `SY10036`; an invalid state is `SY10010`.
2. **Resolve categories.** `GET /sub-categories` returns the sub-category and additional
   category ids. Primary sub-category is mandatory (`SY10024` blank, `SY10025` invalid) and
   you may attach at most nine additional categories (`SY10026`).
3. **Check what the plan syndicates to.** `GET /plan-sites` returns the directory sites
   enabled for the plan. Passing a site id outside that set is `SY10007`.
4. **Create the location.** `createLocation` — `POST /locations`. Watch the field rules
   the catalog encodes: description needs at least 200 characters (`SY10005`), business
   hours must cover all seven days in 12-hour format such as `09:00am` (`SY10031`,
   `SY10033`), and coordinates must agree with the address (`SY10030`).
5. **File it.** `createFolder` (`POST /folders/create`) then `POST /locations/folders` to
   place the location, or `addLocationToTag` (`POST /locations/tags`). A location may carry
   at most ten tags (`SY10037`/`SY10040`) and tags may not contain special characters (`SY10071`).
6. **Add imagery.** `uploadSingleLocationPhoto` (`POST /locations/photos`) accepts JPEG or
   PNG between 10KB and 5MB (`SY10065`, `SY10067`). For bulk uploads poll
   `checkBulkPhotoUploadStatus` (`GET /locations/photos/requests/{requestId}`) — it is a job,
   not a synchronous write. Exactly one logo and one cover image are allowed
   (`SY10060`, `SY10061`).
7. **Confirm.** `listLocations` (`GET /locations`) paginates with `first`/`after` cursors and
   defaults to 50 per page. Read `pageInfo.hasNextPage` and `pageInfo.endCursor`, never a page number.

## What arrives asynchronously

If webhooks are configured for the account you will receive `profile.created`, then
`listing.submission` per directory as syndication lands, and `connection.location_connected`
when a directory profile is linked. Verify `X-Synup-Signature` before trusting any of them.
