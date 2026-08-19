---
name: synup-review-campaign
description: Launch an SMS or email review-request campaign for a Synup location, add customers to it, and follow message status and attributed reviews.
api: Synup API v4
base_url: https://api.synup.com/api/v4
generated: '2026-08-13'
method: generated
source: openapi/synup-api-openapi.yml
operations:
  - createReviewCampaign
  - addCustomersToReviewCampaign
  - listReviewCampaigns
  - listReviewCampaignCustomers
---

# Run a review-request campaign

The high-value shape here is event-triggered: fire a review request the moment a POS marks
an order delivered or a CRM deal closes, then attribute the review that comes back.

## Order of operations

Synup enforces a setup order and tells you so. `SY30019` is *INVALID OPERATION: Construction
Of Campaign Before Construction Of Global Settings* — the account/location global setting
must exist first (`SY30006` when it does not, `SY30020` when you create it twice).

1. **Create the campaign.** `createReviewCampaign` — `POST /locations/review-campaigns`.
   Requires the create-campaign permission (`SY30018`).
2. **Add recipients.** `addCustomersToReviewCampaign` —
   `POST /locations/review-campaigns/customers`. Each customer needs an email or a phone
   (`SY30001`); phone must be numeric (`SY30002`/`SY30033`) and both are deduplicated per
   account (`SY30003`, `SY30034`, `SY30035`). Partial failure is normal: `SY30017`
   (*Validation for some of your customers failed*) means some rows landed and some did not,
   so read the response rather than assuming all-or-nothing.
3. **Sending domain.** Email sending is gated on a verified domain — `SY30048` not verified,
   `SY30055` domain not active, `SY30054` from-email blank, `SY30049` no subdomain associated.
4. **Follow the campaign.** `listReviewCampaigns`
   (`GET /locations/{locationId}/review-campaigns`) and `listReviewCampaignCustomers`
   (`GET /locations/review-campaigns/{reviewCampaignId}/customers`).

## Webhook lifecycle

`campaign.sent` → `campaign.recipients_added` → `campaign.status_change` per message →
`campaign.feedback_submitted` / `campaign.review_posted` (campaign-attributed review) with
`campaign.send_rejected` on failure. Every delivery carries
`X-Synup-Signature: sha256=base64(HMAC-SHA256(secret, raw_body))`; verify it against the raw
bytes with a constant-time compare and answer the `endpoint.verification` challenge before
deliveries will start at all.
