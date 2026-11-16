<p align="center">
  <img src="assets/mandimitra-banner.svg" alt="MandiMitra — Smart Mandi Intelligence and Direct Trade" width="100%" />
</p>

<h1 align="center">MandiMitra</h1>
<p align="center"><strong>Know the market. Choose the right buyer. Trade with confidence.</strong></p>
<p align="center">
  AI-assisted mandi price intelligence · Multilingual voice access · Direct farmer-to-buyer trade
</p>

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/status-Project%20Blueprint-blue" />
  <img alt="Platform" src="https://img.shields.io/badge/platform-Android%20%2B%20Web-brightgreen" />
  <img alt="Backend" src="https://img.shields.io/badge/backend-FastAPI%20%7C%20PostgreSQL-009688" />
  <img alt="ML" src="https://img.shields.io/badge/ML-Price%20Forecasting-orange" />
  <img alt="Languages" src="https://img.shields.io/badge/languages-English%20%7C%20Telugu%20%7C%20Hindi-purple" />
</p>

> **Project stage:** Planning and product specification. This repository currently contains the project blueprint only. No production application, trained forecasting model, verified live integration, or deployed service is claimed to exist yet.

---

## 1. What is MandiMitra?

MandiMitra is a proposed, farmer-first agricultural market intelligence and direct-trading platform for India. It brings three connected capabilities into one experience:

1. **Mandi intelligence:** Explore current and historical market prices and receive short-term, data-backed price estimates for a selected commodity, variety, and mandi.
2. **Voice-first multilingual assistance:** Ask for prices, market comparisons, listing help, and basic trading guidance in a supported language using voice or text.
3. **Direct trade and competitive bids:** Publish a produce lot, receive buyer enquiries and offers, compare bids and estimated net proceeds, and communicate through authenticated messaging.

The goal is not to promise a higher selling price. The goal is to give farmers better information, more buyer reach, and a clearer way to compare selling options.

## 2. The problem

Farmers may need to make selling decisions with fragmented information: prices vary across markets, data can be difficult to interpret, and access to buyers may depend on intermediaries or personal networks. A displayed mandi price alone does not tell a farmer what they may actually receive after transport, handling, quality differences, and buyer terms.

MandiMitra is designed around that decision:

**What can I sell, where can I sell it, what price range is plausible, which buyer offer is better after costs, and how do I contact that buyer safely?**

## 3. Product pillars

| Pillar | What it does | Farmer value |
|---|---|---|
| Market Pulse | Current and historical mandi prices, trends, nearby market comparison | Better visibility into market conditions |
| Price Outlook | Forecast range with confidence and data freshness indicators | A useful estimate, not a guaranteed price |
| Voice Mitra | Speech input, spoken responses, language selection and guided flows | Lower typing and literacy barriers |
| Produce Listings | Create a lot with crop, variety, quantity, quality, photos and availability | Reach buyers beyond existing contacts |
| Buyer Bids | Receive, compare, accept or decline offers | Transparent offer comparison |
| Secure Conversations | Authenticated buyer–farmer chat linked to a listing or bid | Clearer communication and traceability |
| Trust & Safety | Verification status, reporting, dispute records and audit events | More accountable marketplace interactions |

## 4. Primary users

- **Farmers / sellers:** Check markets, understand price trends, list produce and compare offers.
- **Buyers:** Discover produce lots, filter by crop and location, ask questions, and submit bids.
- **Administrators / support staff:** Review reports, manage platform reference data, investigate marketplace abuse and monitor data quality.

The interface must work for people with different levels of digital confidence. Essential actions should use clear language, large touch targets, familiar icons with labels, and voice guidance where supported.

## 5. Proposed experience

~~~mermaid
flowchart LR
    A[Farmer opens app] --> B[Choose language]
    B --> C[Ask by voice or text]
    C --> D[View mandi prices and forecast]
    D --> E[Compare nearby markets]
    E --> F[Create produce listing]
    F --> G[Buyers discover listing]
    G --> H[Buyers submit bids]
    H --> I[Farmer compares net proceeds]
    I --> J[Accept or decline]
    J --> K[Secure conversation and fulfilment]
~~~

## 6. Planned platform and technology

The stack below is a proposed implementation baseline, not a statement about code already present.

| Layer | Proposed choice | Responsibility |
|---|---|---|
| Farmer mobile app | Native Android, Kotlin, Jetpack Compose | Farmer workflows, voice input/output, notifications |
| Buyer / admin web | Next.js, React, TypeScript | Buyer marketplace, operations and administration |
| Application API | Python, FastAPI, Pydantic | Validation, business rules, forecast endpoints and integrations |
| Data platform | PostgreSQL with PostGIS | Users, markets, price observations, listings, bids and geospatial queries |
| Authentication / managed services | Supabase Auth, PostgreSQL and Storage | Identity, database hosting, file storage and access policies |
| Realtime messaging | Supabase Realtime or authenticated WebSocket service | New-message and bid-status updates |
| Forecasting | Python, pandas, scikit-learn; LightGBM/XGBoost after evaluation | Baselines, feature pipelines, model training and inference |
| Notifications | Firebase Cloud Messaging | Push notifications on Android and web where configured |
| Deployment | Docker, managed app hosting and managed database | Reproducible builds, release and operations |

Technology decisions and trade-offs are documented in [ADR-001](docs/decisions/ADR-001-platform-stack.md).

## 7. Data strategy

The initial market-data candidate is the Government of India Open Government Data catalog entry for daily mandi commodity prices generated through AGMARKNET. The catalog describes wholesale minimum, maximum and modal prices; the resource page also exposes variety-wise price data. Integration details, rate limits, API access, field definitions and applicable terms must be verified before implementation.

- [OGD India — Current daily prices of commodities across mandis](https://data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi)
- [OGD India — Variety-wise daily market prices](https://data.gov.in/resource/variety-wise-daily-market-prices-data-commodity)
- [Open-Meteo historical weather API](https://open-meteo.com/en/docs/historical-weather-api) — an optional feature source, subject to terms and suitability for the intended deployment.

Data freshness, missing observations, market/commodity naming, variety, grade and unit differences must be handled explicitly. The app must show the last update time and must never present stale or missing data as live.

## 8. Scope and delivery approach

The project will be developed in stages:

| Phase | Outcome |
|---|---|
| 0 — Foundation | Confirm users, data access, language scope, architecture and acceptance criteria |
| 1 — Market MVP | Farmer onboarding, market search, observed prices, history and comparisons |
| 2 — Forecast MVP | Baseline forecasts, evaluation, forecast ranges and freshness indicators |
| 3 — Marketplace MVP | Produce listings, buyer discovery, bid submission and offer comparison |
| 4 — Voice and trust | Voice-assisted flows, translations, chat, notifications, reporting and verification |
| 5 — Pilot and hardening | Field testing, accessibility, security review, operational monitoring and rollout |

The detailed backlog and exit criteria are in the [implementation roadmap](docs/10-implementation-roadmap.md).

## 9. Documentation map

| Document | Purpose |
|---|---|
| [Project charter](docs/01-project-charter.md) | Vision, outcomes, stakeholders, assumptions and constraints |
| [Product requirements](docs/02-product-requirements.md) | Personas, functional requirements, acceptance criteria and non-functional requirements |
| [System architecture](docs/03-system-architecture.md) | Components, boundaries, data flows and deployment view |
| [Data and ML strategy](docs/04-data-and-ml-strategy.md) | Data sources, ingestion, feature engineering, training and evaluation |
| [Database design](docs/05-database-design.md) | Entity model, table definitions and integrity rules |
| [API contract](docs/06-api-contract.md) | Proposed endpoints, payloads, errors and authorization |
| [UX and accessibility](docs/07-ux-and-accessibility.md) | Navigation, language, voice and primary user journeys |
| [Security and privacy](docs/08-security-and-privacy.md) | Threats, access control, privacy and marketplace safety |
| [Testing strategy](docs/09-testing-and-quality.md) | Test layers, quality gates and acceptance checks |
| [Implementation roadmap](docs/10-implementation-roadmap.md) | Milestones, workstreams, dependencies and release gates |
| [Deployment and operations](docs/11-deployment-and-operations.md) | Environments, CI/CD, monitoring, backups and incident response |
| [Planned repository structure](docs/12-planned-repository-structure.md) | Proposed folder layout and responsibility of each future file |
| [Architecture decision record](docs/decisions/ADR-001-platform-stack.md) | Rationale and alternatives for the initial stack |
| [Contributing guide](CONTRIBUTING.md) | Branches, commits, reviews and documentation standards |

## 10. Success measures

The pilot should measure real outcomes, not vanity metrics:

- Price-data coverage and freshness by market and commodity.
- Forecast error against simple baselines, reported by commodity and horizon.
- Percentage of forecasts that include calibrated uncertainty ranges.
- Successful farmer onboarding and produce-listing completion.
- Buyer response rate and time to first qualified offer.
- Offer comparison completion and listing-to-accepted-offer conversion.
- Task completion and error rates by language and input mode.
- Reports, suspicious activity, unresolved disputes and support response time.

Targets must be set after baseline measurement and farmer/buyer discovery. No income increase or forecast accuracy percentage is assumed in advance.

## 11. Important product safeguards

- Forecasts are estimates, not guaranteed future prices or financial advice.
- Observed prices and model-generated estimates must be visually distinct.
- Every market observation and forecast must show its date, unit, market, variety and source/freshness state where available.
- If data is insufficient, the system should say so rather than invent a forecast.
- A buyer's bid is not a completed sale. Acceptance, fulfilment and payment status must be separate states.
- No payment escrow, lending, insurance or logistics guarantee is part of the first MVP.
- Personal contact details should not be publicly exposed by default.
- Voice and translation failures must have a typed/manual fallback.

## 12. Current status

| Area | Status |
|---|---|
| Product concept and documentation | In this repository |
| UI / mobile application | Not implemented |
| Backend API | Not implemented |
| Database migrations | Not implemented |
| Market-data integration | Not verified or implemented |
| Forecasting model | Not trained or validated |
| Marketplace / chat | Not implemented |
| Deployment | Not configured |

## 13. Getting started with this blueprint

1. Read the [project charter](docs/01-project-charter.md) and [product requirements](docs/02-product-requirements.md).
2. Confirm the first pilot geography, crops, languages and farmer/buyer workflows.
3. Verify access and terms for the market-data source before committing to a forecast scope.
4. Create a small end-to-end vertical slice: one crop, a small set of mandis, observed prices, and one farmer listing.
5. Establish baseline data quality and forecast metrics before describing the model as accurate.
6. Expand only after the pilot meets its usability, safety and quality gates.

---

## Disclaimer

MandiMitra is a proposed software project. It is not affiliated with, endorsed by, or operated by AGMARKNET, the Ministry of Agriculture and Farmers Welfare, e-NAM, or any government agency. External data sources remain subject to their own availability, accuracy, licensing and usage conditions. Market prices and forecasts can change, and the platform must not guarantee a sale price or farmer income.

