# ADR-001 — Initial Platform and Technology Choices

- **Status:** Proposed baseline for the MVP; revisit after discovery and a small technical spike.
- **Date:** 2026-11-16
- **Decision owners:** Product and engineering

## Context

MandiMitra needs an Android-first farmer experience, a buyer/admin web portal, a backend for marketplace state and source integrations, relational records for market prices and bids, and a Python-friendly workflow for data quality and forecasting.

The first release should be implementable by a small team. It should avoid unnecessary infrastructure while keeping provider integrations, model code and database access replaceable.

## Decision

Use the following baseline unless discovery or technical validation reveals a material blocker:

| Concern | Proposed choice | Rationale |
|---|---|---|
| Farmer mobile app | Kotlin + Jetpack Compose | Native Android UX, device speech/notification support and strong control over low-connectivity behavior |
| Buyer/admin web | Next.js + React + TypeScript | Responsive web marketplace and operations portal |
| Backend API | Python + FastAPI + Pydantic | Typed APIs and shared ecosystem with data ingestion and ML |
| Primary database | PostgreSQL | Relational constraints and transactions for listings, bids and messages |
| Geospatial | PostGIS | Market and district proximity queries |
| Managed platform | Supabase Auth, PostgreSQL, Storage and Realtime may be used | Faster pilot operations while retaining PostgreSQL portability |
| Forecasting | pandas + scikit-learn baselines; evaluate LightGBM/XGBoost | Start with simple measurable baselines and progress only when data supports it |
| Notifications | Firebase Cloud Messaging where appropriate | Push delivery for Android and supported web clients |
| Packaging | Docker for backend and worker services | Reproducible local/staging environments |

## Boundary decision

All sensitive business operations must pass through a trusted backend or a rigorously designed database policy. Client applications may use provider-approved public configuration, but no privileged database service key or data-provider secret may be shipped in a client.

If Supabase is selected:
- Supabase Auth may manage identity/session issuance.
- PostgreSQL remains the durable data store.
- RLS provides defense in depth for client-facing data access.
- The FastAPI backend owns privileged ingestion, forecast, marketplace and state-transition logic.
- Storage buckets remain private by default for non-public files.
- Realtime subscriptions enforce participant authorization.

Do not build two competing sources of truth for identity or marketplace state.

## Alternatives considered

| Alternative | Advantages | Trade-offs / reason not selected initially |
|---|---|---|
| React Native for farmer app | Shared TypeScript skills and cross-platform potential | Native Android is the initial target; device integration and offline behavior should be evaluated before choosing a cross-platform layer |
| Flutter | Consistent cross-platform UI and strong widget system | Adds a Dart stack alongside Python/TypeScript; no clear first-pilot requirement for iOS |
| Firebase as full backend | Integrated auth, database, hosting and notifications | Relational price/bid workflows, constraints and analytical queries are a natural fit for PostgreSQL |
| Custom PostgreSQL hosting | Direct control and portability | More database operations, backup and security work for a small pilot |
| Node.js API | Shared language with Next.js | Python has a simpler shared path for ingestion, data processing and model workflows |
| Separate microservices from day one | Independent scaling and deployment | Operational overhead without evidence of scale or isolation needs |
| Deep learning for initial forecast | Flexible model class | Requires sufficient clean history and careful evaluation; must first beat simple baselines |

## Consequences

### Positive

- Clear separation between Android farmer UX and buyer/admin web.
- Strong relational consistency for bids and listing transitions.
- Shared Python ecosystem for source ingestion and forecasting.
- Managed infrastructure can reduce initial operational burden.
- API and provider adapters preserve future migration options.

### Costs and trade-offs

- Multiple languages and build systems must be maintained.
- Native Android and web need separate UI implementation.
- Supabase and FastAPI require explicit decisions about which service owns each operation.
- RLS policies and backend authorization need careful testing to prevent inconsistent permissions.
- Managed services have provider-specific limits and pricing that must be checked before production.

## Validation tasks

Before implementation is locked:
1. Build a minimal Android sign-in and language-selection spike.
2. Build a minimal web buyer-listing spike.
3. Verify the chosen auth provider's token validation flow with FastAPI.
4. Verify RLS behavior for farmer, buyer and administrator roles.
5. Fetch and inspect a representative permitted market-data sample.
6. Test one database migration and a transactional bid state change.
7. Confirm the deployment and data retention requirements for the pilot.

## Revisit this decision when

- iOS support becomes a launch requirement.
- Android device compatibility or speech support is inadequate.
- Market-data scale requires separate ingestion infrastructure.
- Model inference needs independent scaling.
- Supabase limits, data residency or operational constraints no longer fit.
- A field pilot demonstrates that the current UX or architecture is too expensive or difficult to use.
