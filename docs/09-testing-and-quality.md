# 09 — Testing and Quality Strategy

## 1. Quality objectives

The system must be safe to use for market discovery and marketplace communication. Correct numbers, correct units, correct permissions and truthful states matter more than visual polish alone.

Quality goals:
- Observed price records match source meaning and unit.
- Forecast evaluation is reproducible and time-aware.
- Users cannot read or modify unauthorized marketplace data.
- Bid and listing transitions remain consistent under concurrent actions.
- Core workflows are usable in supported languages and target devices.
- Offline, empty, stale and failure states are understandable.

## 2. Test pyramid

~~~mermaid
flowchart TD
  A[Manual field and usability validation] --> B[End-to-end tests]
  B --> C[API and integration tests]
  C --> D[Unit and property tests]
  D --> E[Static analysis and schema checks]
~~~

Run fast unit and static checks on every change. Run integration and end-to-end tests in CI and before release. Field testing is required before a pilot because device, language and connectivity behavior cannot be fully simulated.

## 3. Test layers

| Layer | What to test | Example |
|---|---|---|
| Unit | Pure business rules, validation, conversions and calculations | Quintal/kg conversion, bid expiry, listing state guard |
| Component | Android and web components in isolation | Price card labels observed vs estimated |
| API | Request/response validation, auth, error behavior | Buyer cannot read seller's private listing details |
| Database | Constraints, transactions and RLS | Two buyers cannot both be accepted for one indivisible lot |
| Integration | Data provider adapter, storage, messaging and notifications | Partial ingestion is logged without deleting prior observations |
| ML | Feature correctness, leakage prevention and evaluation | Training split uses only data available at forecast time |
| End-to-end | Complete user journeys | Farmer searches market, lists produce and accepts a bid |
| Accessibility | Screen reader, contrast, scaling and focus order | Farmer completes primary flow without color-only cues |
| Field / usability | Real device, local language and poor network | Farmer uses voice or text to find price and create a listing |

## 4. Critical test scenarios

### Market data

- Missing modal price.
- Source row with inconsistent minimum and maximum.
- Wrong or missing unit.
- Duplicate row from repeated ingestion.
- Late-arriving observation.
- Unknown market/commodity alias.
- Feed unavailable or stale.
- Provider schema changes.
- Partial page/API retrieval.
- Currency/unit mismatch in comparison.

### Forecasting

- Sparse market history.
- Commodity with no historical data.
- New market or unseen category.
- Missing weather or arrivals feature.
- Future-data leakage attempt.
- Seasonal split and rolling-origin backtest.
- Baseline model beats candidate model.
- Model artifact unavailable.
- Prediction interval fails expected coverage.
- Source data changes after model training.
- Delayed actual observation for monitoring.

### Marketplace

- Farmer creates, edits, pauses and closes own lot.
- Buyer cannot edit another buyer's bid.
- Bid expires before farmer accepts.
- Farmer closes listing while a bid is being submitted.
- Two buyers attempt bid acceptance at the same time.
- Partial-quantity bid and lot remainder.
- Bid currency/unit mismatch.
- Listing photo upload interrupted.
- Moderation rejects a listing.
- Report submitted and resolved by authorized admin.

### Communication and security

- Non-participant cannot load a conversation.
- Realtime subscription cannot bypass participant checks.
- Message send retries do not duplicate messages.
- Push token belongs to the authenticated user.
- Private pickup details not exposed in public listing results.
- Revoked user session cannot continue protected actions.
- Rate limits respond with a safe retry message.

## 5. ML evaluation tests

- Assert every feature timestamp is less than or equal to the prediction issue time.
- Assert train/validation/test windows do not overlap improperly.
- Assert each evaluation segment has sample count and coverage recorded.
- Assert the model is compared with declared baselines.
- Assert interval coverage and pinball loss are reported for quantile outputs.
- Assert no metric is reported when there are insufficient test observations.
- Assert a model release cannot be marked approved without an evaluation artifact.
- Assert inference returns unavailable if the requested target falls outside approved model scope.

## 6. Data quality tests

| Check | Expected result |
|---|---|
| Minimum price <= maximum price | Valid rows pass; invalid rows are quarantined/flagged |
| Currency and unit present | Missing required values are rejected |
| Canonical market exists | Unknown market enters mapping review |
| Duplicate source record | Idempotent upsert; no duplicate observation |
| Source timestamp older than freshness limit | Result marked stale |
| No record for a date | No synthetic observation is created |
| Source correction | Revision/provenance is retained |
| Variety mismatch | Comparison is blocked or explicitly qualified |

## 7. Performance and resilience

Set budgets after measuring pilot infrastructure. At minimum test:
- Price search with typical and upper-bound date ranges.
- Listing search with combined filters.
- Conversation history pagination.
- Bid submission and acceptance under concurrent requests.
- Cold start and model artifact loading.
- Provider rate limits and retries.
- Database connection loss.
- Temporary storage or push provider failure.
- Mobile app behavior on slow and interrupted connections.

Use bounded retries with jitter for transient provider errors. Avoid retry storms and duplicate business actions.

## 8. CI quality gates

Recommended pull-request checks:

1. Format and lint checks.
2. Type checks and static analysis.
3. Unit tests.
4. API schema/contract tests.
5. Database migration validation.
6. Security and dependency scan.
7. Build Android and web artifacts.
8. Integration tests against isolated test services.
9. ML feature/evaluation checks when model-related files change.

Protect production deployments with a human approval and a successful test run.

## 9. Test environments and data

| Environment | Data | Purpose |
|---|---|---|
| Local | Synthetic or sanitized fixtures | Developer feedback |
| CI | Deterministic synthetic fixtures and test accounts | Repeatable automated checks |
| Staging | Sanitized, representative data | Release rehearsal and integration tests |
| Pilot/production | Real operational data under approved terms | Controlled real-world operation |

Never copy production personal data into developer machines or public fixtures. Synthetic farmer identities, listing examples and price observations must be labelled as synthetic.

## 10. Release checklist

- [ ] All P0 acceptance tests pass.
- [ ] Role and RLS tests pass with independent accounts.
- [ ] Price source, unit and freshness labels are visible.
- [ ] Forecast release report is approved or forecast is disabled.
- [ ] Bid/listing concurrency tests pass.
- [ ] Accessibility checks cover primary tasks.
- [ ] Offline, stale, empty and error states are tested.
- [ ] No critical/high unresolved security issue remains.
- [ ] Backup and restore path has been tested.
- [ ] Support and incident owners are named.
