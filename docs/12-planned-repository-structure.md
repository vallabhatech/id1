# 12 — Planned Repository Structure and File Reference

## 1. Purpose

This document describes the target monorepo layout for the future implementation. It is a **proposed structure**, not a claim that these application files already exist.

The repository currently contains the project README, visual banner and planning documents. Implementation directories should be added as the corresponding work begins, rather than creating empty folders for every future idea.

## 2. Target directory tree

~~~text
id1/
├── README.md
├── CONTRIBUTING.md
├── assets/
│   └── mandimitra-banner.svg
├── docs/
│   ├── 01-project-charter.md
│   ├── 02-product-requirements.md
│   ├── 03-system-architecture.md
│   ├── 04-data-and-ml-strategy.md
│   ├── 05-database-design.md
│   ├── 06-api-contract.md
│   ├── 07-ux-localization-accessibility.md
│   ├── 08-security-privacy-safety.md
│   ├── 09-testing-and-quality.md
│   ├── 10-implementation-roadmap.md
│   ├── 11-deployment-and-operations.md
│   ├── 12-planned-repository-structure.md
│   └── decisions/
│       └── ADR-001-platform-stack.md
├── apps/
│   ├── farmer-android/
│   │   ├── app/
│   │   ├── core/
│   │   ├── feature-auth/
│   │   ├── feature-market/
│   │   ├── feature-listings/
│   │   ├── feature-bids/
│   │   ├── feature-messages/
│   │   └── gradle/
│   └── buyer-web/
│       ├── app/
│       ├── components/
│       ├── features/
│       ├── lib/
│       ├── public/
│       └── tests/
├── services/
│   ├── api/
│   │   ├── app/
│   │   ├── migrations/
│   │   └── tests/
│   ├── ingestion/
│   │   ├── adapters/
│   │   ├── jobs/
│   │   └── tests/
│   └── forecasting/
│       ├── data/
│       ├── features/
│       ├── models/
│       ├── evaluation/
│       └── tests/
├── contracts/
│   ├── openapi/
│   └── events/
├── infra/
│   ├── docker/
│   └── environments/
├── supabase/
│   ├── migrations/
│   └── seed/
└── .github/
    └── workflows/
        └── ci.yml
~~~

## 3. Existing documentation files

| Path | Responsibility |
|---|---|
| README.md | Public project overview, current status, feature pillars and documentation navigation |
| assets/mandimitra-banner.svg | Original project banner used by the README |
| docs/01-project-charter.md | Vision, mission, scope, outcomes, stakeholders and risks |
| docs/02-product-requirements.md | User roles, journeys, functional and non-functional requirements |
| docs/03-system-architecture.md | Components, trust boundaries, data flows and deployment view |
| docs/04-data-and-ml-strategy.md | Source candidates, ingestion, quality, features, evaluation and forecast safeguards |
| docs/05-database-design.md | Entity model, table responsibilities, constraints, indexes and RLS intent |
| docs/06-api-contract.md | Proposed endpoints, payloads, error format and concurrency rules |
| docs/07-ux-localization-accessibility.md | Screen map, voice interactions, translations and accessibility |
| docs/08-security-privacy-safety.md | Threat model, identity, data classification, abuse controls and incident response |
| docs/09-testing-and-quality.md | Test layers, critical scenarios, CI gates and release checklist |
| docs/10-implementation-roadmap.md | Phases, deliverables, dependencies and exit criteria |
| docs/11-deployment-and-operations.md | Environments, CI/CD, ingestion operations, monitoring and recovery |
| docs/12-planned-repository-structure.md | Target source tree and file-by-file responsibility |
| docs/decisions/ADR-001-platform-stack.md | Initial technology decision and alternatives |
| CONTRIBUTING.md | Contribution flow, review expectations and quality standards |

## 4. Planned Android application files

| Planned path | Responsibility |
|---|---|
| apps/farmer-android/settings.gradle.kts | Gradle project and module inclusion |
| apps/farmer-android/build.gradle.kts | Root plugin and build configuration |
| apps/farmer-android/gradle/libs.versions.toml | Central dependency and plugin versions |
| apps/farmer-android/app/src/main/AndroidManifest.xml | Application metadata and declared permissions |
| apps/farmer-android/app/src/main/kotlin/.../MainActivity.kt | Android entry point and Compose host |
| apps/farmer-android/core/model/ | Typed app/domain models |
| apps/farmer-android/core/network/ | API client, token attachment and network error mapping |
| apps/farmer-android/core/database/ | Optional local cache and draft persistence |
| apps/farmer-android/core/designsystem/ | Shared themes, typography and reusable controls |
| apps/farmer-android/feature-auth/ | Sign-in, onboarding, role and language selection |
| apps/farmer-android/feature-market/ | Market search, observed prices, charts and forecast presentation |
| apps/farmer-android/feature-listings/ | Farmer lot creation, media and listing management |
| apps/farmer-android/feature-bids/ | Bid inbox, bid detail and offer comparison |
| apps/farmer-android/feature-messages/ | Conversations and report actions |
| apps/farmer-android/app/src/main/res/values/strings.xml | Default interface strings |
| apps/farmer-android/app/src/main/res/values-te/strings.xml | Telugu translations after review |
| apps/farmer-android/app/src/main/res/values-hi/strings.xml | Hindi translations after review |

Use package names and module boundaries that the team can maintain; do not split modules just to mirror this plan if the implementation remains very small.

## 5. Planned buyer web files

| Planned path | Responsibility |
|---|---|
| apps/buyer-web/package.json | Web dependencies and scripts |
| apps/buyer-web/next.config.ts | Next.js configuration |
| apps/buyer-web/tsconfig.json | TypeScript compiler configuration |
| apps/buyer-web/src/app/layout.tsx | Root layout, providers and metadata |
| apps/buyer-web/src/app/page.tsx | Buyer entry/discovery page |
| apps/buyer-web/src/app/listings/ | Listing search and detail routes |
| apps/buyer-web/src/app/bids/ | Buyer bid management routes |
| apps/buyer-web/src/app/admin/ | Protected administration routes |
| apps/buyer-web/src/components/ | Reusable UI components |
| apps/buyer-web/src/features/ | Feature-specific UI and client logic |
| apps/buyer-web/src/lib/api.ts | Typed API client |
| apps/buyer-web/src/lib/auth.ts | Session and role-aware route helpers |
| apps/buyer-web/public/ | Static icons, fonts and public assets |
| apps/buyer-web/tests/ | Component and browser tests |

## 6. Planned API service files

| Planned path | Responsibility |
|---|---|
| services/api/pyproject.toml | Python project metadata and dependencies |
| services/api/Dockerfile | API container build |
| services/api/app/main.py | FastAPI application creation and router registration |
| services/api/app/core/config.py | Typed environment configuration |
| services/api/app/core/security.py | Token verification and authorization helpers |
| services/api/app/api/v1/router.py | Versioned route aggregation |
| services/api/app/api/v1/markets.py | Market and commodity reference endpoints |
| services/api/app/api/v1/prices.py | Observed price and series endpoints |
| services/api/app/api/v1/forecasts.py | Forecast query endpoint |
| services/api/app/api/v1/listings.py | Listing operations |
| services/api/app/api/v1/bids.py | Bid and offer operations |
| services/api/app/api/v1/messages.py | Conversation and message operations |
| services/api/app/schemas/ | Pydantic request and response models |
| services/api/app/domain/ | Domain rules and state transitions |
| services/api/app/repositories/ | Database access and query definitions |
| services/api/app/services/ | Business services and provider orchestration |
| services/api/migrations/ | Version-controlled schema migrations |
| services/api/tests/ | Unit, API, database and integration tests |

## 7. Planned ingestion files

| Planned path | Responsibility |
|---|---|
| services/ingestion/pyproject.toml | Worker dependencies and scripts |
| services/ingestion/adapters/base.py | Common provider adapter interface |
| services/ingestion/adapters/ogd_agmarknet.py | Approved OGD/AGMARKNET data adapter after access verification |
| services/ingestion/normalize.py | Canonical market, commodity and unit mapping |
| services/ingestion/validate.py | Structural, statistical and temporal validation |
| services/ingestion/jobs/daily_market_prices.py | Scheduled price ingestion job |
| services/ingestion/jobs/reconcile.py | Late/corrected source record reconciliation |
| services/ingestion/tests/ | Adapter, mapping, idempotency and bad-row tests |

## 8. Planned forecasting files

| Planned path | Responsibility |
|---|---|
| services/forecasting/config.py | Model and feature configuration |
| services/forecasting/data/build_dataset.py | Reproducible training dataset creation |
| services/forecasting/features/price_features.py | Lag and rolling-price features |
| services/forecasting/features/calendar_features.py | Calendar and seasonal features |
| services/forecasting/baselines.py | Last-value, seasonal-naive and rolling baselines |
| services/forecasting/train.py | Model training entry point |
| services/forecasting/evaluate.py | Rolling-origin evaluation and metrics |
| services/forecasting/predict.py | Versioned inference interface |
| services/forecasting/registry.py | Model metadata and approval status |
| services/forecasting/tests/ | Leakage, feature, metric and inference tests |

## 9. Planned contracts and infrastructure files

| Planned path | Responsibility |
|---|---|
| contracts/openapi/openapi.yaml | Generated/reviewed HTTP API contract |
| contracts/events/market-data-ingested.json | Internal ingestion event schema if event messaging is introduced |
| contracts/events/bid-status-changed.json | Bid transition event schema if event messaging is introduced |
| infra/docker/docker-compose.yml | Local development services |
| infra/environments/ | Non-secret environment templates and deployment notes |
| supabase/migrations/ | Supabase/PostgreSQL schema and policy migrations if Supabase is selected |
| supabase/seed/ | Synthetic reference data for local development |
| .github/workflows/ci.yml | Lint, tests, scans and build checks |

## 10. File placement rules

- UI belongs in the platform app, not in the API service.
- Domain rules belong in reusable service/domain modules, not duplicated in screens.
- External provider parsing belongs in adapters.
- Canonical mappings belong in controlled reference data.
- Training/evaluation code is separate from API request handlers.
- Migrations are versioned and reviewed.
- Secrets never belong in the repository.
- Tests live close to their owning application/service or in a clearly named test suite.
- Generated artifacts are not committed unless a documented release process requires them.
