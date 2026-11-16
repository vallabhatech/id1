# 10 — Implementation Roadmap

## 1. Delivery strategy

Build MandiMitra as a sequence of small, demonstrable vertical slices. First prove that reliable market observations can be retrieved and shown correctly. Then validate forecasting. Build marketplace workflows with transactional state handling. Add voice, localization depth and operational hardening before expanding pilot coverage.

This roadmap defines dependencies and exit criteria, not a guaranteed calendar schedule. Team size, data access and field-partner availability determine actual duration.

## 2. Phase overview

| Phase | Theme | Primary deliverable | Exit gate |
|---|---|---|---|
| 0 | Discovery and foundation | Validated scope, pilot geography, data-source decision, architecture and project setup | Data access and user workflows are understood |
| 1 | Market information MVP | Searchable mandi observations, history and freshness labels | Source fidelity and core price journey pass |
| 2 | Forecast pilot | Baseline and candidate model, backtest report and cautious forecast UI | Model demonstrates value over baseline or forecast remains disabled |
| 3 | Direct marketplace | Produce listings, buyer search, bids and seller offer inbox | Listing/bid state and authorization tests pass |
| 4 | Voice, language and trust | Voice-assisted queries, translations, chat, notifications and reports | Primary tasks pass language and safety testing |
| 5 | Controlled pilot and scale readiness | Field validation, monitoring, support, backups and release runbook | Pilot gate signed off; known limitations communicated |

## 3. Phase 0 — Discovery and foundation

### Workstreams

- Interview representative farmers, buyers and field partners.
- Select a bounded first geography and a small crop/variety set.
- Verify OGD/AGMARKNET data access, schema, update cadence and terms.
- Inspect market naming, units, duplicate rates, gaps and historical coverage.
- Confirm initial languages and whether speech-to-text is viable on target devices.
- Define user roles, public/private listing fields and moderation policy.
- Create design system, repo conventions, environment plan and CI skeleton.
- Agree success metrics and baseline measurement plan.

### Deliverables

- Pilot scope and user research summary.
- Data-source verification note and sample data dictionary.
- Approved architecture decision record.
- API/database first-pass contracts.
- Clickable wireframes or tested screen flows.
- Threat model and privacy data map.
- Issue backlog with dependencies.

### Exit criteria

- One pilot crop and market can be traced from source to normalized record.
- Source access and intended use have been reviewed.
- Farmer and buyer core tasks are agreed.
- No unresolved critical ambiguity in identity, listing visibility or bid acceptance rules.

## 4. Phase 1 — Market information MVP

### Workstreams

**Data:** Build source adapter, ingestion run tracking, quality checks, canonical market/crop mapping and repeatable historical retrieval where permitted.

**Backend:** Implement reference endpoints, price search, time-series API, cache/freshness metadata and access-safe public routes.

**Android/web:** Language selection, crop/market search, results, price detail and historical trend.

**Quality:** Validate source rows, unit presentation, date handling and stale-feed behavior.

### Deliverables

- Working ingestion for the agreed pilot dataset.
- Searchable observed prices with source and timestamp.
- Historical series and market comparison where data is compatible.
- Empty, partial, stale and failure states.
- Test fixtures and data-quality report.

### Exit criteria

- Sampled displayed values reconcile to source records.
- Repeated ingestion does not create duplicates.
- Unit, market, commodity and date are visible.
- Stale or missing observations are not presented as current values.

## 5. Phase 2 — Forecast pilot

### Workstreams

- Build reproducible feature pipeline.
- Establish last-observation, seasonal-naive and rolling-median baselines.
- Implement time-based rolling-origin backtests.
- Evaluate candidate tabular model only if data volume/quality permits.
- Measure MAE, WAPE/sMAPE as appropriate, interval coverage and sample counts.
- Build a model registry/version manifest and inference contract.
- Add forecast UI with a clear estimate/range label and limitation state.

### Deliverables

- Data/feature validation suite.
- Baseline and model evaluation report by crop, market and horizon.
- Versioned model artifact and reproducible training metadata if approved.
- Forecast endpoint and UI, or an explicit no-forecast state if model criteria fail.

### Exit criteria

A model is enabled only if it beats the agreed baseline on approved evaluation segments, has measured uncertainty behavior, avoids leakage and passes product/domain review. A negative model result is a valid outcome; the team should continue with observed data rather than force a forecast feature.

## 6. Phase 3 — Direct marketplace

### Workstreams

- Implement farmer profile and listing draft/publish workflow.
- Add controlled media upload and private/public location projections.
- Implement buyer listing discovery and filters.
- Implement bid submission, expiry, withdrawal, edit and status history.
- Add farmer bid inbox and offer comparison.
- Implement transactional acceptance and listing-state locking.
- Add audit events for important state changes.

### Deliverables

- End-to-end farmer listing flow.
- Buyer discovery and bid flow.
- Farmer bid review, accept/decline and offer comparison.
- Listing/bid moderation controls and report entry point.

### Exit criteria

- No unauthorized cross-user access.
- Bid unit/currency/quantity are explicit.
- Concurrent acceptance race tests pass.
- UI never labels accepted bid as delivered or paid.
- Public responses do not leak private pickup or contact details.

## 7. Phase 4 — Voice, localization and trust

### Workstreams

- Implement device/service speech adapter with permission handling.
- Support Telugu, Hindi and English interface strings after human review.
- Add confirmed intent extraction for a constrained set of queries.
- Implement text fallback for every voice workflow.
- Add listing-linked conversations and participant access checks.
- Add in-app and push notification flows.
- Implement reports, moderation queue, user support and block/report controls.
- Add accessibility and low-connectivity improvements.

### Deliverables

- Voice query for agreed intents with editable recognized text.
- Language switch and reviewed translations.
- Authenticated chat and bid notifications.
- Report/resolve process and moderation audit history.

### Exit criteria

- Key tasks work without microphone permission.
- Critical strings and consent copy are reviewed by language speakers.
- Speech mistakes cannot silently submit bids or listings.
- Realtime access cannot bypass conversation membership.
- Support and moderation owners are prepared for pilot.

## 8. Phase 5 — Controlled pilot and hardening

### Workstreams

- Deploy isolated staging and production environments.
- Configure production secrets, monitoring, alerts and backups.
- Conduct privacy and security review.
- Run user acceptance sessions on real devices and variable networks.
- Train support staff and document escalation.
- Instrument service, ingestion and forecast health.
- Roll out to a bounded cohort and review feedback before expansion.

### Deliverables

- Release candidate and deployment runbook.
- Security review findings and remediation log.
- Backup/restore evidence.
- Pilot support and incident process.
- Pilot outcome report, including limitations and negative results.

### Exit criteria

- Critical/high security issues are resolved.
- Restore procedure is tested.
- Data freshness and failure alerts are active.
- Users can report issues and receive support.
- Product claims are supported by pilot measurements.

## 9. Suggested work breakdown

| Workstream | Initial tasks | Depends on |
|---|---|---|
| Product and UX | Personas, workflows, wireframes, language glossary | Discovery |
| Mobile | Compose app shell, auth, market screens, listing flow | API contract and UX |
| Web | Buyer search, bid forms, admin review | API contract and roles |
| Backend | Auth verification, schema, endpoints, state transitions | Requirements and DB design |
| Data | Provider adapter, normalization, quality checks, ingestion | Source access and schema |
| ML | Baselines, features, backtest, inference | Reliable historical data |
| Security | Threat model, RLS, audit, secrets, abuse controls | Architecture and schema |
| QA | Fixtures, unit/integration/e2e tests, field validation | Each workstream increment |
| Operations | Environments, CI/CD, metrics, backups, runbooks | Hosting and release model |

## 10. Definition of Ready

A backlog item is ready when:
- User/problem and desired outcome are clear.
- Acceptance criteria are testable.
- Dependencies and data requirements are known.
- Security/privacy implications are identified.
- UI/API/schema impact is understood.
- Scope fits a reviewable implementation increment.

## 11. Definition of Done

A backlog item is done when:
- Code and tests are complete.
- Authorization, error and empty states are covered.
- API/schema documentation is updated.
- UI is localized and accessible where applicable.
- Logging does not expose sensitive data.
- Review has been completed and CI passes.
- Any feature flag or migration is documented.
- User-facing behavior matches the requirement.

## 12. Initial priority order

1. Verify market-data source and create a reproducible ingestion sample.
2. Build one end-to-end observed-price journey.
3. Establish data quality and market/crop mapping.
4. Build baseline forecast evaluation, then decide if ML adds value.
5. Deliver farmer listing and buyer bid workflows.
6. Add chat, notifications, voice and wider language coverage.
7. Pilot, measure and harden.

This order avoids investing heavily in voice or complex models before the product has trustworthy underlying data and a validated core workflow.
