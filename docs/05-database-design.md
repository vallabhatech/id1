# 05 — Database Design

## 1. Design principles

- PostgreSQL is the proposed system of record.
- Use UUID primary keys for application-owned entities and preserve external provider IDs separately.
- Store money and quantities in fixed-precision numeric columns.
- Store event timestamps in UTC using timezone-aware columns.
- Keep canonical market observations separate from user-created listings and buyer bids.
- Preserve source provenance and ingestion history.
- Apply database constraints, transactions and row-level security (RLS) in addition to API authorization.
- Keep personally identifying data out of public market/listing queries.

## 2. Entity relationship overview

~~~mermaid
erDiagram
  AUTH_USERS ||--|| PROFILES : has
  PROFILES ||--o{ PRODUCE_LISTINGS : owns
  MARKET_LOCATIONS ||--o{ PRICE_OBSERVATIONS : records
  COMMODITIES ||--o{ PRICE_OBSERVATIONS : describes
  COMMODITIES ||--o{ PRODUCE_LISTINGS : describes
  COMMODITIES ||--o{ FORECASTS : predicts
  MARKET_LOCATIONS ||--o{ FORECASTS : targets
  INGESTION_RUNS ||--o{ PRICE_OBSERVATIONS : imports
  PRODUCE_LISTINGS ||--o{ LISTING_MEDIA : contains
  PRODUCE_LISTINGS ||--o{ BIDS : receives
  PROFILES ||--o{ BIDS : submits
  PRODUCE_LISTINGS ||--o{ CONVERSATIONS : contextualizes
  CONVERSATIONS ||--o{ CONVERSATION_PARTICIPANTS : includes
  PROFILES ||--o{ CONVERSATION_PARTICIPANTS : joins
  CONVERSATIONS ||--o{ MESSAGES : contains
  PROFILES ||--o{ MESSAGES : sends
  PROFILES ||--o{ REPORTS : creates
  PROFILES ||--o{ NOTIFICATIONS : receives
  PROFILES ||--o{ AUDIT_EVENTS : performs
~~~

The diagram is conceptual. The final migration may include additional lookup tables, constraints, indexes and provider-specific staging tables.

## 3. Core tables

### 3.1 Identity and reference data

| Table | Important columns | Purpose |
|---|---|---|
| profiles | id PK/FK to auth user, role, display_name, preferred_locale, district_id, created_at, updated_at | Application profile and role; identity credentials stay with the auth provider |
| market_locations | id PK, source_market_id, market_name, state_id, district_id, latitude, longitude, location_point, active | Canonical mandi references and geospatial search |
| states | id PK, state_code, name | State lookup |
| districts | id PK, state_id FK, district_code, name | District lookup |
| commodities | id PK, source_commodity_id, canonical_name, category, default_unit, active | Canonical crop/commodity list |
| commodity_varieties | id PK, commodity_id FK, source_variety_id, name, active | Variety reference where supported |
| measurement_units | code PK, display_name, dimension, conversion_group | Controlled quantity and price units |

### 3.2 Market data and model output

| Table | Important columns | Purpose |
|---|---|---|
| ingestion_runs | id PK, provider, resource_id, adapter_version, started_at, finished_at, status, received_count, accepted_count, rejected_count, warnings, checksum | Provenance and operational history for source ingestion |
| price_observations | id PK, market_id FK, commodity_id FK, variety_id nullable FK, observed_on, min_price, max_price, modal_price, currency, price_unit, arrivals_quantity nullable, arrivals_unit nullable, source_name, source_record_id, retrieved_at, ingestion_run_id FK, quality_status | Source-reported observed mandi prices |
| price_observation_revisions | id PK, observation_id FK, previous_values, revised_values, reason, changed_by, changed_at | Traceable corrections when source records require adjustment |
| forecasts | id PK, market_id FK, commodity_id FK, variety_id nullable FK, target_date, horizon, estimate, lower_bound, upper_bound, currency, price_unit, model_name, model_version, generated_at, data_through_date, quality_status, limitation_reason | Versioned forecast output, separate from observations |
| model_registry | id PK, model_name, model_version, artifact_uri, training_start, training_end, feature_version, metrics, approval_status, created_at | Model artifacts, evaluation and release state |

### 3.3 Marketplace

| Table | Important columns | Purpose |
|---|---|---|
| produce_listings | id PK, seller_id FK, commodity_id FK, variety_id nullable FK, description, quantity, unit, quality_notes, asking_price nullable, price_unit nullable, availability_from, market_id nullable FK, district_id, public_location_label, private_pickup_reference nullable, status, published_at, expires_at, created_at, updated_at | Farmer-declared lot |
| listing_media | id PK, listing_id FK, storage_path, media_type, size_bytes, sort_order, created_at | Private or policy-controlled photo references |
| bids | id PK, listing_id FK, buyer_id FK, offered_price, currency, price_unit, quantity, quantity_unit, validity_until, terms, status, created_at, updated_at | Structured buyer offer |
| bid_events | id PK, bid_id FK, actor_id FK, from_status, to_status, reason, created_at | Bid transition history |
| conversations | id PK, listing_id FK, bid_id nullable FK, created_at, last_message_at, status | Conversation context |
| conversation_participants | conversation_id FK, user_id FK, joined_at, last_read_at | Participant membership and read cursor |
| messages | id PK, conversation_id FK, sender_id FK, body, created_at, edited_at nullable, deleted_at nullable | Message history; attachments can be separate controlled records |

### 3.4 Trust, notifications and audit

| Table | Important columns | Purpose |
|---|---|---|
| reports | id PK, reporter_id FK, target_type, target_id, category, description, status, assigned_to nullable FK, resolution, created_at, resolved_at | User reports and moderation cases |
| notifications | id PK, recipient_id FK, type, title_key, body_data, target_type, target_id, read_at, created_at | In-app notification records |
| audit_events | id PK, actor_id nullable FK, action, target_type, target_id, metadata, request_id, created_at | Traceability for important administrative and marketplace actions |
| device_tokens | id PK, user_id FK, token_hash or protected_token, platform, active, last_seen_at | Push token registry; access is restricted |

## 4. Important constraints

### Price observations

- min_price, max_price and modal_price are nullable only when the upstream dataset legitimately omits a field.
- When all are present, min_price must be less than or equal to max_price.
- modal_price outside the min/max interval is flagged for review rather than automatically rewritten.
- currency and price_unit are mandatory.
- A uniqueness strategy must prevent duplicate source observations. Prefer provider source ID; otherwise define a normalized composite key that handles nullable variety and grade fields.
- Source values and normalized values should both be traceable.

### Listings

- quantity must be greater than zero.
- unit must be a supported unit.
- asking_price is optional; the farmer may choose to receive bids without publishing a target price.
- published listings require all mandatory fields and must pass moderation rules where configured.
- a closed listing cannot receive new active bids.
- public listing data must not include private pickup details.

### Bids

- offered_price and quantity must be greater than zero.
- bid currency, price unit and quantity unit are explicit.
- validity_until must be later than creation time.
- only an eligible buyer can submit or update their own bid.
- accepting a bid must run in a transaction that locks or conditionally updates the listing and checks current state.
- accepted, declined, withdrawn, expired and superseded states are distinct.

### Conversations

- each conversation has a defined listing context.
- participants are derived from the seller and eligible buyer/bid relationship.
- every read and send operation checks membership.
- deleted or moderated content follows a documented retention policy; it is not silently removed from audit context where retention is required.

## 5. Access-control matrix

| Resource | Anonymous | Farmer owner | Buyer participant | Administrator |
|---|---|---|---|---|
| Public market observations | Read public fields | Read | Read | Read/manage references |
| Profile | Limited public fields only | Read/update own allowed fields | Read limited public fields | Restricted support view |
| Listing | Published public fields | Full own listing access | Published fields | Moderation access |
| Bid | None or limited public state | Read bids for own listing; accept/decline | Read/update own bids | Moderation access |
| Conversation | None | If participant | If participant | Restricted investigation with audit |
| Message | None | If participant | If participant | Restricted investigation with audit |
| Report | Submit allowed report | Read own submission status where policy allows | Same | Review/resolve |
| Audit events | None | Own visible event summaries only if needed | Same | Privileged audited access |

## 6. RLS policy patterns

These are policy intentions, not ready-to-run SQL.

- profiles: users may read/update only their own private profile fields; public profile view exposes a restricted projection.
- produce_listings: public users read published listings only; seller can manage own drafts and listings.
- bids: buyer reads and manages own bids; seller reads bids on own listing and performs allowed state transitions.
- conversations/messages: participant membership is required for every select/insert.
- reports: reporter can submit; only authorized moderation role can update resolution.
- notifications: recipient can read and mark their own notifications.
- audit_events: users cannot mutate or delete audit events; privileged readers have controlled access.

Do not rely on client-side filtering. Test RLS using multiple users and all role combinations.

## 7. Indexing strategy

| Table | Candidate index | Query served |
|---|---|---|
| price_observations | (market_id, commodity_id, observed_on DESC) | Market/commodity time series |
| price_observations | (commodity_id, observed_on DESC, market_id) | Commodity comparison |
| price_observations | (source_name, source_record_id) unique where present | Ingestion idempotency |
| forecasts | (market_id, commodity_id, target_date DESC, generated_at DESC) | Latest forecast lookup |
| produce_listings | (status, commodity_id, published_at DESC) | Published lot search |
| produce_listings | (district_id, commodity_id, status) | Location and crop filters |
| bids | (listing_id, status, created_at DESC) | Seller bid inbox |
| bids | (buyer_id, status, created_at DESC) | Buyer's own bid list |
| messages | (conversation_id, created_at DESC) | Conversation history |
| conversation_participants | (user_id, conversation_id) | User's conversations |
| reports | (status, created_at DESC) | Moderation queue |

Use PostGIS spatial indexes for geography queries after storing the actual point/geography representation. Add indexes only after reviewing query plans and actual pilot usage.

## 8. Data retention and privacy

- Do not store authentication passwords in application tables.
- Keep exact pickup location private until the product's agreed disclosure point.
- Limit access to device tokens and private image objects.
- Define retention periods for messages, reports, audit events and user deletion requests before pilot.
- Use soft deletion only where necessary and document what remains in audit records.
- Avoid logging access tokens, message contents, phone numbers or exact coordinates.
- Establish backups and restoration tests.

