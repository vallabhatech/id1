# 02 — Product Requirements Document

**Status:** Proposed requirements for MVP and staged expansion  
**Platforms:** Android farmer app and responsive buyer/admin web portal

## 1. Product summary

MandiMitra connects market intelligence to the selling workflow. Farmers explore observed mandi prices and carefully labelled forecasts, publish produce lots, receive structured bids and communicate with buyers. Buyers discover produce and submit offers. Administrators maintain reference data and review reports.

## 2. Roles and permissions

| Role | Capabilities | Restrictions |
|---|---|---|
| Farmer | View market data, manage own profile/lots, view bids, accept or decline bids, listing-linked chat | Cannot edit another farmer's lot or another buyer's bid |
| Buyer | Search published lots, submit own bids, manage own bids, listing-linked chat | Cannot view private farmer data or alter another buyer's bid |
| Administrator | Review reports, moderate content/accounts, manage reference data and audit records | Privileged actions require authorization and audit history |
| Support agent (optional) | Review cases and limited account context | No unrestricted administration or database access |

## 3. Personas

### Farmer / seller
Needs to know which nearby market has relevant prices, what the available data says, and whether a buyer offer is reasonable after transport and other known costs. May prefer Telugu voice prompts and simple screens.

### Produce buyer
Needs to find lots by crop, variety, location, expected availability, quantity and quality notes. Needs clear offer terms and a reliable follow-up channel.

### Administrator / support operator
Needs to identify suspicious listings, resolve reports, maintain market and commodity references, and investigate data freshness or service incidents.

## 4. Core user journeys

1. Farmer selects a language and completes account setup.
2. Farmer searches a commodity and nearby market.
3. Farmer reviews observed prices, history and any available forecast range.
4. Farmer creates a produce lot with confirmed quantity, unit, availability date and location.
5. Buyer discovers the lot and submits a structured bid.
6. Farmer compares bids, including known deductions and estimated logistics.
7. Farmer accepts or declines a bid.
8. Participants communicate in the listing context and coordinate fulfilment. Payment and transport are not assumed to be provided by the MVP.

## 5. Functional requirements

Priority: **P0** = first useful MVP; **P1** = next increment; **P2** = later enhancement.

### Identity and onboarding

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-ID-01 | P0 | Register and sign in | Successful login creates a valid session; invalid credentials show a safe error |
| FR-ID-02 | P0 | Select preferred language | Choice persists and can be changed |
| FR-ID-03 | P0 | Select user role | Role-restricted actions require server-side authorization |
| FR-ID-04 | P1 | Profile supports display name, district and language | Private fields are not exposed publicly |
| FR-ID-05 | P1 | Account recovery | Recovery does not reveal whether an unrelated account exists |

### Market intelligence

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-MKT-01 | P0 | Search by state, district, mandi, commodity and date | Results show filters and observation date |
| FR-MKT-02 | P0 | Show min/max/modal price when available | Each value includes currency, unit, market and date |
| FR-MKT-03 | P0 | Show source and last-updated time | Timestamp comes from ingestion metadata |
| FR-MKT-04 | P0 | Show historical trend | Chart labels date and unit; missing dates are not fabricated |
| FR-MKT-05 | P1 | Compare multiple mandis | Only compatible commodity, variety, grade and units are compared |
| FR-MKT-06 | P1 | Filter variety and grade when available | Unsupported dimensions are labelled unavailable, not guessed |

### Forecasting

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-ML-01 | P1 | Estimate short-horizon prices for supported pairs | Response includes model version, generation time, horizon and data-through date |
| FR-ML-02 | P1 | Show uncertainty interval | UI distinguishes estimate and interval from observed price |
| FR-ML-03 | P1 | Display data-quality state | Sparse or stale data can return unavailable rather than fabricated output |
| FR-ML-04 | P1 | Compare model to baselines | Evaluation records MAE and suitable scale-aware metrics by horizon and segment |
| FR-ML-05 | P2 | Explain major forecast drivers | Explanation is traceable to available features and does not imply causation |

### Voice and language

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-VCE-01 | P1 | Accept voice input for supported intents | Recognized text is shown for confirmation before consequential action |
| FR-VCE-02 | P1 | Read key prices and instructions aloud | Speech follows selected language where available |
| FR-VCE-03 | P1 | Provide text fallback | Essential task can be completed without microphone access |
| FR-VCE-04 | P1 | Translate interface and validation messages | Critical action is not available only in untranslated text |
| FR-VCE-05 | P2 | Extract intent from natural-language market query | Extracted crop, place and date are shown and editable before search |

### Marketplace

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-LST-01 | P0 | Farmer creates produce listing | Required fields validate; listing begins in draft or pending-review state |
| FR-LST-02 | P0 | Farmer publishes, edits, pauses or closes own listing | Only owner or authorized moderator changes listing state |
| FR-LST-03 | P0 | Buyer searches published listings | Filters include crop, location, availability and quantity where available |
| FR-BID-01 | P0 | Buyer submits structured bid | Bid contains price, price unit, quantity, expiry/validity and terms |
| FR-BID-02 | P0 | Farmer views bids for own listing | Bid terms are visible according to privacy rules |
| FR-BID-03 | P0 | Farmer accepts or declines a bid | State transition is atomic; two bids cannot be accepted for one indivisible lot |
| FR-BID-04 | P1 | Farmer compares estimated net proceeds | Included costs are itemized; unknown costs are labelled excluded |
| FR-BID-05 | P1 | Eligible bid can be withdrawn or revised | History is retained and transitions follow defined rules |

### Communication and trust

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-COM-01 | P1 | Authenticated listing-linked chat | Only eligible participants can read or send |
| FR-COM-02 | P1 | Notifications for bids and state changes | Notification opens the correct protected resource |
| FR-TRU-01 | P1 | Report listing, bid, message or account | Report stores category, optional description and target reference |
| FR-TRU-02 | P1 | Administrator reviews and resolves reports | Every moderation decision records actor, time and reason |
| FR-TRU-03 | P2 | Display verification status | Label states exactly what was checked and by whom |

## 6. Non-functional requirements

| ID | Area | Requirement / target-setting method |
|---|---|---|
| NFR-PER-01 | Performance | Set p95 latency budgets after pilot scale is known; prioritize search and listing screens |
| NFR-AVL-01 | Availability | Define MVP service-level objectives before pilot; show degraded states when feeds fail |
| NFR-SEC-01 | Security | Enforce authorization server-side and through database policies |
| NFR-PRV-01 | Privacy | Minimize collection, explain usage and define retention/deletion processes |
| NFR-A11Y-01 | Accessibility | Large touch targets, readable contrast, screen-reader labels, scalable text and non-voice paths |
| NFR-LOC-01 | Localization | Format currency, date, number and unit by locale while retaining canonical stored values |
| NFR-REL-01 | Reliability | Use retry-safe mutations where feasible and audit important state changes |
| NFR-DAT-01 | Data quality | Validate units, dates, mappings, duplicates and source freshness |
| NFR-OBS-01 | Observability | Structured logs, error tracking, latency metrics and pipeline health |
| NFR-MNT-01 | Maintainability | Versioned API contracts, migrations, tests and decision records |

## 7. Business rules

1. A price observation is a source-reported record; it is not a forecast.
2. A forecast is an estimate for a defined horizon and carries model version and data context.
3. A listing is a seller-declared lot, not certified quality unless a certification workflow exists.
4. A bid is an offer with stated terms; it is not a completed sale.
5. Accepted bid, fulfilment, delivery and payment are separate states.
6. Net-proceeds comparison includes only declared/configured costs; unknown costs remain visible.
7. Timestamps are stored in UTC and formatted for the selected locale.
8. Users may mutate only resources they own or are authorized to manage.
9. No forecast is generated when minimum history and quality checks fail.
10. Translation must not alter numeric values, units or offer terms.

## 8. MVP acceptance gate

The MVP is eligible for a controlled pilot only when:

- Authentication and role authorization pass security tests.
- Pilot market data is ingested with documented provenance and freshness.
- Observed prices show source date, market, commodity, variety where available, unit and currency.
- Farmers can publish, edit and close their own lots.
- Buyers can submit structured bids and farmers can accept or decline them safely.
- Core workflows pass on target Android devices and common mobile browsers.
- Error, empty, offline and stale-data states are understandable.
- Reports can be submitted and reviewed by an authorized operator.
- A privacy notice, user support path and incident owner are established.

## 9. Privacy-conscious analytics

Suggested events: onboarding_completed, language_changed, market_search_submitted, market_result_viewed, forecast_viewed, listing_draft_saved, listing_published, bid_submitted, bid_viewed, bid_accepted, bid_declined, chat_message_sent and report_submitted.

Do not include phone numbers, message bodies, exact home coordinates or other sensitive personal details in analytics payloads. Analytics must be disclosed and configured with appropriate consent and retention settings.
