# 06 — Proposed API Contract

**Status:** Design contract only. Routes and payloads must be implemented and versioned before they are treated as live interfaces.

## 1. API conventions

- Base path: /api/v1
- Transport: HTTPS only.
- Payload format: JSON using UTF-8.
- Authentication: bearer access token issued by the configured identity provider.
- Authorization: verified by the server for every protected resource.
- IDs: UUIDs for application-owned resources.
- Timestamps: ISO 8601 UTC strings.
- Currency: explicit ISO currency code; initial pilot currency is INR.
- Pagination: cursor-based for feeds and larger result sets.
- Errors: stable machine-readable code plus safe human-readable message.
- Mutations: validate input and use transaction boundaries for bid/listing state transitions.

Do not expose database credentials, provider API keys, model artifact paths or internal stack traces in responses.

## 2. Authentication and profile

| Method | Path | Access | Purpose |
|---|---|---|---|
| GET | /me | Authenticated | Return current user's permitted profile |
| PATCH | /me | Authenticated | Update allowed profile fields |
| PUT | /me/language | Authenticated | Update preferred locale |
| GET | /reference/languages | Public | List enabled interface languages |
| GET | /reference/roles | Public | Return supported role descriptions, not permission grants |

Role assignment and any verification status must be validated server-side. A client cannot elevate its own role by sending a role value in a profile update.

## 3. Market intelligence

| Method | Path | Access | Purpose |
|---|---|---|---|
| GET | /markets | Public | Search market references by state/district/name |
| GET | /commodities | Public | Search canonical commodities |
| GET | /commodities/{commodity_id}/varieties | Public | List supported varieties |
| GET | /prices | Public | Search observed mandi prices |
| GET | /prices/series | Public | Retrieve a historical series |
| GET | /markets/compare | Public | Compare compatible market observations |
| GET | /forecasts | Public or authenticated per abuse policy | Retrieve a forecast when supported |

### Example price request

~~~http
GET /api/v1/prices?market_id=8f70c8c0-1c7b-4f3f-9cc0-6d31e3d56001&commodity_id=7b83f2c7-d14b-4dfd-8f93-69bb48d4d102&from=2026-10-01&to=2026-10-15
~~~

### Example observed-price response

~~~json
{
  "data": [
    {
      "observation_id": "7bf1c8e6-bc4c-4c1e-9df9-2e7e0b3d65ab",
      "market": {
        "id": "8f70c8c0-1c7b-4f3f-9cc0-6d31e3d56001",
        "name": "Example Market",
        "district": "Example District",
        "state": "Telangana"
      },
      "commodity": {
        "id": "7b83f2c7-d14b-4dfd-8f93-69bb48d4d102",
        "name": "Example Commodity"
      },
      "observation_date": "2026-10-15",
      "min_price": "1800.00",
      "max_price": "2200.00",
      "modal_price": "2000.00",
      "currency": "INR",
      "price_unit": "quintal",
      "source": "Configured market data provider",
      "retrieved_at": "2026-10-16T04:30:00Z",
      "freshness": "current"
    }
  ],
  "page": {
    "next_cursor": null
  }
}
~~~

The example values are illustrative only and must never be seeded as real market observations.

### Example forecast response

~~~json
{
  "data": {
    "market_id": "8f70c8c0-1c7b-4f3f-9cc0-6d31e3d56001",
    "commodity_id": "7b83f2c7-d14b-4dfd-8f93-69bb48d4d102",
    "target_date": "2026-10-18",
    "horizon_days": 3,
    "estimate": "2050.00",
    "lower_bound": "1880.00",
    "upper_bound": "2210.00",
    "currency": "INR",
    "price_unit": "quintal",
    "model_name": "pilot-modal-price-model",
    "model_version": "0.1.0",
    "generated_at": "2026-10-15T18:00:00Z",
    "data_through_date": "2026-10-15",
    "quality_status": "pilot",
    "disclaimer": "Estimate only; actual trading prices may differ."
  }
}
~~~

The forecast example is illustrative, not a live prediction. The service must return unavailable when data checks or model approval fail.

## 4. Produce listings

| Method | Path | Access | Purpose |
|---|---|---|---|
| POST | /listings | Farmer | Create a draft lot |
| GET | /listings | Public | Search published listings |
| GET | /listings/{listing_id} | Public / owner | Read permitted lot details |
| PATCH | /listings/{listing_id} | Owner | Update an eligible draft/published lot |
| POST | /listings/{listing_id}/publish | Owner | Submit for moderation/publication |
| POST | /listings/{listing_id}/pause | Owner | Pause active listing |
| POST | /listings/{listing_id}/close | Owner | Close listing |
| GET | /me/listings | Farmer | List current user's lots |

### Example create listing request

~~~json
{
  "commodity_id": "7b83f2c7-d14b-4dfd-8f93-69bb48d4d102",
  "variety_id": null,
  "quantity": "18.00",
  "quantity_unit": "quintal",
  "quality_notes": "Harvested this week; grading details to be confirmed",
  "availability_from": "2026-11-20",
  "market_id": "8f70c8c0-1c7b-4f3f-9cc0-6d31e3d56001",
  "public_location_label": "Example District"
}
~~~

The example describes a fictional listing. Private pickup coordinates or contact details should use a separate protected workflow.

## 5. Bids and offer comparison

| Method | Path | Access | Purpose |
|---|---|---|---|
| POST | /listings/{listing_id}/bids | Buyer | Submit bid |
| GET | /listings/{listing_id}/bids | Listing owner | View offers received |
| GET | /me/bids | Buyer | View own submitted bids |
| PATCH | /bids/{bid_id} | Buyer owner | Revise eligible bid |
| POST | /bids/{bid_id}/withdraw | Buyer owner | Withdraw eligible bid |
| POST | /bids/{bid_id}/accept | Listing owner | Accept offer transactionally |
| POST | /bids/{bid_id}/decline | Listing owner | Decline offer |
| GET | /listings/{listing_id}/offer-comparison | Listing owner | Compare compatible offers and known costs |

Bid state transitions must be validated against the current database state. An API response must not imply that the sale is completed, delivered or paid merely because a bid was accepted.

### Example bid request

~~~json
{
  "offered_price": "2150.00",
  "currency": "INR",
  "price_unit": "quintal",
  "quantity": "10.00",
  "quantity_unit": "quintal",
  "validity_until": "2026-11-18T12:00:00Z",
  "terms": "Subject to agreed quality inspection and weighing."
}
~~~

## 6. Messaging, reports and notifications

| Method | Path | Access | Purpose |
|---|---|---|---|
| GET | /conversations | Participant | List own conversations |
| GET | /conversations/{id}/messages | Participant | Read conversation history |
| POST | /conversations/{id}/messages | Participant | Send a message |
| POST | /reports | Authenticated | Report a listing, bid, message or account |
| GET | /me/notifications | Authenticated | List own notifications |
| POST | /notifications/{id}/read | Recipient | Mark notification read |
| POST | /devices/push-tokens | Authenticated | Register a push token |
| DELETE | /devices/push-tokens/{id} | Owner | Disable a push token |

Realtime subscriptions must enforce the same participant policies as the HTTP API.

## 7. Standard error format

~~~json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields are invalid.",
    "request_id": "req_01JEXAMPLE",
    "details": [
      {
        "field": "quantity",
        "reason": "Must be greater than zero."
      }
    ]
  }
}
~~~

Recommended status codes:

| HTTP status | Meaning |
|---|---|
| 400 | Invalid request syntax or unsupported query |
| 401 | Missing or invalid authentication |
| 403 | Authenticated but not authorized |
| 404 | Resource does not exist or is intentionally hidden |
| 409 | State conflict, duplicate or stale version |
| 422 | Field validation failed |
| 429 | Rate limit exceeded |
| 500 | Unexpected server failure |
| 503 | Required dependency unavailable |

## 8. Concurrency and idempotency

- State-changing requests should accept an idempotency key where duplicate submission could create multiple records.
- Bid acceptance must use a database transaction and conditional state update.
- Listing updates should use a version number or updated-at precondition to detect stale edits.
- Realtime messages and push notifications are not proof of a committed business action.
- Return a request ID for support and trace correlation.

## 9. API security checklist

- Verify token signature, issuer, audience and expiry.
- Authorize ownership and role for every protected route.
- Validate request size, content type and all identifiers.
- Apply rate limits to authentication-adjacent, search, chat, report and upload endpoints.
- Do not leak private contact or pickup details in public responses.
- Avoid returning raw exception strings or SQL details.
- Log operational metadata without logging tokens or message bodies.
