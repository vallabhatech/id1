# 03 — System Architecture

## 1. Architecture goals

The architecture supports a bounded pilot that can grow without making the first release unnecessarily complex.

- Keep farmer workflows mobile-first and resilient to inconsistent connectivity.
- Separate observed market data from forecasts and marketplace records.
- Keep business rules and authorization on trusted server/database boundaries.
- Make external data providers replaceable through adapters.
- Track provenance for observations and model outputs.
- Support an Android client and responsive web portal.
- Never expose database service credentials or privileged keys in clients.

## 2. Proposed technology baseline

| Layer | Proposed technology | Responsibility |
|---|---|---|
| Farmer application | Kotlin, Jetpack Compose | Native Android experience, device speech and notifications |
| Buyer/admin portal | Next.js, React, TypeScript | Responsive web workflows |
| API | FastAPI, Python, Pydantic | Validation, integrations, business rules and inference |
| Primary database | PostgreSQL | Durable relational marketplace and market records |
| Geospatial support | PostGIS | Market proximity and location filtering |
| Managed platform option | Supabase Auth, PostgreSQL, Storage, Realtime | Reduce pilot operations while retaining access controls |
| Forecast stack | pandas, scikit-learn; LightGBM/XGBoost after evaluation | Feature pipelines, training and serving |
| Notifications | Firebase Cloud Messaging | Push delivery for configured clients |
| Packaging | Docker | Reproducible API and worker environments |

The first implementation can keep forecast code in the Python backend package. Split it into a separate inference service only when scale, deployment isolation or dependency requirements justify it.

## 3. Logical architecture

~~~mermaid
flowchart TB
  subgraph Clients
    A[Android Farmer App]
    B[Buyer and Admin Web Portal]
  end
  subgraph Trusted["Trusted application boundary"]
    C[HTTPS API / FastAPI]
    D[Authentication and Authorization]
  end
  subgraph Services
    E[Market Data Adapter]
    F[Forecast Module]
    G[Marketplace Module]
    H[Messaging and Notifications]
    I[Admin and Audit Module]
  end
  subgraph Data
    J[(PostgreSQL + PostGIS)]
    K[(Private Object Storage)]
    L[(Model Artifacts and Metrics)]
  end
  subgraph External["External providers"]
    M[OGD India / AGMARKNET-derived data]
    N[Weather Provider]
    O[Push Provider]
    P[Device Speech Services]
  end
  A --> C
  B --> C
  A --> P
  C --> D
  C --> E
  C --> F
  C --> G
  C --> H
  C --> I
  E --> M
  F --> N
  E --> J
  F --> J
  F --> L
  G --> J
  G --> K
  H --> J
  H --> O
  I --> J
~~~

## 4. Component responsibilities

| Component | Responsibilities | Must not do |
|---|---|---|
| Android app | Language selection, farmer screens, voice capture, draft UX and push display | Hold service-role keys or decide authorization |
| Buyer/admin web | Listing search, bids, operational screens | Trust client-supplied role or ownership claims |
| FastAPI | Authentication verification, validation, rate limits and business rules | Expose raw privileged database access |
| Market adapter | Fetch approved source data, map source fields, preserve provenance | Invent missing values or units |
| Ingestion worker | Schedule fetches, deduplicate, validate and record runs | Rewrite prior observations without an audit trail |
| Forecast module | Build features, train/evaluate and emit versioned estimates | Present unvalidated estimates as guaranteed outcomes |
| Marketplace module | Listings, bids, state transitions and comparison calculations | Mark fulfilment or payment without an implemented verified workflow |
| Messaging module | Authorize participants, store messages and dispatch notifications | Expose private contact fields by default |
| Admin module | Reference data, moderation, reports and audit views | Permit unaudited privileged changes |
| PostgreSQL | Durable records, relationships, constraints and transactional state | Store raw secrets or unnecessary personal data |
| Object storage | Listing photos and permitted attachments | Make private files public without an explicit policy |

## 5. Main data flows

### 5.1 Market data ingestion

~~~mermaid
sequenceDiagram
  participant S as Scheduler
  participant W as Ingestion Worker
  participant P as Provider Adapter
  participant D as Source Dataset
  participant DB as PostgreSQL
  S->>W: Start scheduled ingestion
  W->>P: Request configured date / market range
  P->>D: Fetch approved dataset/API
  D-->>P: Source records
  P-->>W: Normalized rows + source metadata
  W->>W: Validate dates, units, names and duplicates
  W->>DB: Save observations and ingestion run
  W->>DB: Save rejected-row counts and warnings
~~~

Every run stores provider name, resource identifier, retrieval time, source date range, row counts, rejected-row counts and a checksum or equivalent provenance marker.

### 5.2 Price request

1. Client submits commodity, market, date range and supported variety/grade filters.
2. API validates identifiers and requested range.
3. Market query returns observations with source date and unit.
4. If a forecast is requested, the forecast module checks model availability and input coverage.
5. Response returns observed and forecast data in separate typed sections.
6. Client labels stale, unavailable or low-confidence results.

### 5.3 Listing and bid lifecycle

~~~mermaid
stateDiagram-v2
  [*] --> Draft
  Draft --> PendingReview: Submit
  PendingReview --> Published: Approved
  PendingReview --> Rejected: Rejected
  Rejected --> Draft: Edit and resubmit
  Published --> Paused: Seller pauses
  Paused --> Published: Seller resumes
  Published --> Closed: Seller closes
  Published --> Awarded: Seller accepts a bid
  Awarded --> Closed: Outcome recorded
  Closed --> [*]
~~~

Acceptance must be transactional. Two bids cannot simultaneously be marked accepted for the same indivisible lot. Split lots require an explicit allocation model.

### 5.4 Messaging

Messages belong to a conversation associated with a listing and eligible participants. The server checks participant membership for every read and send operation. Realtime delivery is an optimization; durable message storage remains the source of truth.

## 6. Data boundaries and trust model

- Clients are untrusted; validate and authorize every request server-side.
- Authentication proves a session; authorization decides what that session can do.
- Database row-level security is defense in depth, not a replacement for API checks.
- Privileged database keys remain only in backend secrets.
- External provider payloads are untrusted input and must be schema-validated.
- Uploaded images need size/type limits and safe processing.
- Forecast outputs are advisory and must carry lineage.

## 7. Deployment view

~~~mermaid
flowchart LR
  U[Users] --> CDN[Web Hosting / CDN]
  U --> APP[Android Distribution]
  CDN --> API[API Service]
  APP --> API
  API --> DB[(Managed PostgreSQL)]
  API --> OBJ[(Private Object Storage)]
  API --> Q[Background Job Queue]
  Q --> ING[Ingestion Worker]
  Q --> ML[Forecast Worker]
  ING --> DB
  ML --> DB
  API --> PUSH[Push Provider]
  ING --> EXT[Approved Data Providers]
  ML --> WX[Weather Provider]
~~~

A managed queue can be deferred for the first vertical slice. A scheduled worker is acceptable if retries, run history and idempotency are implemented.

## 8. API and persistence boundaries

- API versioning starts at v1.
- JSON request/response contracts use typed models.
- IDs are opaque UUIDs unless an external provider requires source identifiers.
- Timestamps are stored as UTC timestamps.
- Prices always include a currency and unit reference.
- Quantities use decimal-safe storage, not binary floating-point database columns.
- Market observations are append-oriented and retain source identifiers.
- Listing and bid changes use explicit state transitions and audit events.

## 9. Scalability strategy

### MVP
- One FastAPI deployment.
- One managed PostgreSQL database.
- One scheduled ingestion worker.
- Forecasting in the Python service or a separate worker process.
- Managed object storage and push service.

### Growth
- Split ingestion and prediction workers from request-serving API.
- Add queue-based retries and dead-letter handling.
- Add read replicas or materialized views when measured query volume justifies them.
- Cache reference data and popular market summaries.
- Partition large observation tables only after measuring query and maintenance needs.
- Isolate ML training from online inference.
- Add regional infrastructure only when latency, data residency or availability requirements justify it.

## 10. Failure handling

| Failure | Expected behavior |
|---|---|
| Price provider unavailable | Keep last successful data with visible timestamp; mark feed stale |
| Partial ingestion | Record partial status and row counts; do not erase valid prior observations |
| Forecast artifact missing | Return forecast unavailable; keep observed prices available |
| Speech service unavailable | Show typed search and manual form |
| Realtime channel disconnects | Show stored history and reconnect safely |
| Push delivery fails | Persist in-app notification state and log delivery failure |
| Image upload interrupted | Keep listing draft and allow retry |
| Database unavailable | Return temporary error; do not falsely confirm mutations |

## 11. Architecture decisions to revisit

- Whether Supabase Realtime is sufficient for message scale.
- Whether forecasting needs a separately deployed service.
- Which approved data and weather providers are viable under real access terms.
- Whether a queue is needed for the first pilot.
- Whether the admin portal should be separate from the buyer portal.
