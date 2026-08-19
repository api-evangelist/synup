---
name: synup-review-response-desk
description: Pull reviews and Q&A across every connected directory for a set of Synup locations, draft and post owner responses, and track review analytics.
api: Synup API v4
base_url: https://api.synup.com/api/v4
generated: '2026-08-13'
method: generated
source: openapi/synup-api-openapi.yml
operations:
  - GET /locations/{locationId}/reviews
  - listInteractionsByIds
  - GET /rollup_interactions
  - respondToInteraction
  - getAnalytics
  - GET /reviews/site-config
---

# Run a review response desk

Synup calls a review or a Q&A item an **interaction**. One feed carries every connected
directory — Google, Facebook, TripAdvisor, Trustpilot and the rest — so you respond once
against Synup rather than once per platform.

## Steps

1. **Know which sources are live.** `GET /reviews/site-config` lists the interaction
   sources configured for the account; `GET /locations/{location_id}/reviews/settings`
   narrows it to one location. An unknown source name is `SY50104`.
2. **Pull the feed.** `GET /locations/{locationId}/reviews` fetches interactions for a
   location. For a multi-location sweep use `GET /rollup_interactions`. Ratings filters
   must be within 1–5 (`SY50110`) and must not repeat (`SY50118`); duplicate location ids
   are `SY50119`.
3. **Hydrate specific items.** `listInteractionsByIds` (`GET /reviewDetails`) takes the
   interaction ids you selected. Interaction ids are UUID strings, not integers.
4. **Respond.** `respondToInteraction` (`POST /locations/reviews/respond`). Response content
   must be non-empty (`SY50103`). **Google reviews can only be replied to once** — `SY50121`.
   Treat that as a hard, unrecoverable state, not a retry.
5. **Edit or withdraw.** The same operationId backs `POST /locations/reviews/respond/edit`
   and `POST /locations/reviews/respond/archive`. Synup publishes `respondToInteraction`
   three times over three different paths, so bind by path, not by operationId.
6. **Measure.** `getAnalytics` (`GET /locations/{locationId}/{type}`) returns interaction
   analytics for a location.

## Guardrails

- No idempotency key exists. A retried respond call posts a second reply where the platform
  allows it. Persist the interaction id you have already answered.
- `SY50300` (*Response not successful: Received status code 500*) is a downstream directory
  failure surfaced verbatim — retry with backoff; it is not a request error.
- Webhooks `interaction.review` and `interaction.response` fire on new reviews and new
  responses. They are location-scoped and social-category interactions are not delivered.
