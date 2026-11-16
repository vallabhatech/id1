# 11 — Deployment and Operations Plan

**Status:** Planned operating model; no environment or deployment is configured by this document.

## 1. Environment strategy

| Environment | Purpose | Data | Access |
|---|---|---|---|
| Local development | Feature implementation and debugging | Synthetic fixtures or approved sanitized samples | Individual developers |
| CI | Automated build, test and security checks | Deterministic synthetic fixtures | CI service identity |
| Staging | Integration, UAT and release rehearsal | Sanitized representative data | Team and invited testers |
| Pilot / production | Controlled real-user service | Approved operational data | Restricted operators and end users |

Never copy production personal data into local development or public test fixtures.

## 2. Proposed services

| Service | Deployment responsibility |
|---|---|
| Android application | Signed Android build distributed through the chosen controlled channel |
| Buyer/admin web | Next.js build hosted on a managed web platform |
| FastAPI | Containerized HTTPS API service |
| Ingestion worker | Scheduled container or managed job |
| Forecast worker | Training/evaluation job and versioned inference artifact |
| PostgreSQL | Managed PostgreSQL, optionally Supabase |
| Object storage | Private bucket for listing images and permitted documents |
| Push notifications | Firebase Cloud Messaging where configured |
| Observability | Structured logs, error tracking, metrics and alerts |

Keep the first deployment small. Do not introduce Kubernetes, a complex event bus or multiple microservices unless measured requirements justify them.

## 3. Configuration and secrets

Create environment-specific configuration for:
- database connection and migration identity;
- public auth-provider URL and public client key where applicable;
- backend-only service credentials;
- token verification configuration;
- approved market-data provider credentials and resource identifiers;
- optional weather provider credentials;
- private object storage configuration;
- push notification credentials;
- log/monitoring endpoints;
- allowed web origins and application base URLs.

Secrets must live in the hosting provider's secret manager. Do not place secret values in source control, screenshots, issue comments or client-side environment variables. Use different credentials for local, staging and production.

## 4. Database migration process

1. Create reviewed, version-controlled migrations.
2. Validate migrations against a clean database in CI.
3. Apply them to staging before production.
4. Back up the production database before high-risk schema changes.
5. Use backward-compatible expand/migrate/contract changes for tables used by active clients.
6. Verify constraints, RLS policies and indexes after migration.
7. Document rollback or forward-fix steps for every production migration.

Never make untracked manual schema changes in production.

## 5. CI/CD pipeline

~~~mermaid
flowchart TD
  A[Pull request] --> B[Format / lint / type checks]
  B --> C[Unit and contract tests]
  C --> D[Security and dependency scans]
  D --> E[Build mobile, web and API]
  E --> F[Integration tests]
  F --> G{All required checks pass?}
  G -- No --> H[Block merge and fix]
  G -- Yes --> I[Review and merge]
  I --> J[Deploy to staging]
  J --> K[Smoke tests and UAT]
  K --> L{Release approved?}
  L -- No --> M[Hold release]
  L -- Yes --> N[Deploy controlled pilot]
  N --> O[Monitor and evaluate]
~~~

Suggested pipeline controls:
- Protected main branch.
- Required pull-request review.
- Required CI checks.
- No direct production deployment from an unreviewed branch.
- Deployment approval for production.
- Artifact version and source commit recorded per release.
- Database migration step clearly separated from application deployment.

## 6. Scheduled data ingestion operations

- Run the provider adapter on a schedule consistent with source update cadence.
- Store every run and its outcome.
- Retry transient errors with bounded exponential backoff and jitter.
- Stop retries for non-retryable schema/authentication errors and alert maintainers.
- Detect incomplete pagination and partial downloads.
- Compare new row counts and coverage with recent runs.
- Mark affected market data stale when the feed exceeds the accepted freshness window.
- Reconcile late-arriving and corrected observations without losing provenance.

The refresh schedule must follow the verified source behavior; do not claim real-time data if the provider publishes daily records.

## 7. Monitoring and alerting

| Signal | Why it matters | Example alert condition |
|---|---|---|
| API availability and latency | User-facing health | Sustained error rate or p95 regression |
| Authentication failures | Abuse or configuration issue | Unexpected spike |
| Database health | Core system dependency | Connection exhaustion or storage pressure |
| Ingestion freshness | Price reliability | Latest successful run exceeds configured threshold |
| Ingestion quality | Source/schema change | Rejection rate or field-null rate spikes |
| Forecast health | Model availability | Inference errors or artifact missing |
| Forecast performance | Decision usefulness | Error/interval coverage degrades after actuals arrive |
| Listing/bid errors | Marketplace reliability | Failed state transitions increase |
| Message delivery | Communication reliability | Repeated send or realtime failures |
| Reports and moderation queue | User safety | High-severity case exceeds response target |

Alerts need an owner, severity, runbook and escalation path. Avoid alerting on every transient provider error.

## 8. Backup and recovery

- Enable managed database backups appropriate to the pilot's recovery objectives.
- Define recovery point objective (RPO) and recovery time objective (RTO) before real-user launch.
- Include database and required object-storage metadata in recovery planning.
- Test restoration into an isolated environment.
- Protect backups with access controls and encryption.
- Retain model artifacts, evaluation reports and source-data lineage required for reproducibility.
- Document how to rotate credentials after a restore or suspected exposure.

A backup that has never been restored is not a verified recovery plan.

## 9. Release and rollback

Before a release:
- Confirm migration compatibility.
- Confirm feature flags and disabled fallback behavior.
- Verify provider and model configuration.
- Run smoke tests for login, market search, listing, bids and messaging.
- Record build version, commit and model version.
- Have a rollback or forward-fix owner.

If a model regresses, disable forecast display without disabling observed market prices. If a provider feed is unavailable, show the last valid data with a stale label or suppress it according to freshness policy.

## 10. Incident runbooks to create

| Runbook | Required content |
|---|---|
| Market feed unavailable | Provider status, recent successful run, stale-data behavior, retry and escalation |
| Market feed schema changed | Sample response, adapter impact, quarantine plan and mapping update |
| Forecast degradation | Model version, segment metrics, rollback/disable path |
| Authentication incident | Session revocation, credential rotation and user communication |
| Suspected data exposure | Containment, evidence preservation, impact assessment and notification review |
| Marketplace fraud report | Triage, listing/bid review, account actions and appeal route |
| Database restore | Backup selection, isolated restore, verification and cutover |
| Push/realtime outage | In-app fallback and delivery reconciliation |

## 11. Operational ownership

Before pilot, name a primary and backup owner for:
- application/API incidents;
- database and backups;
- data ingestion and provider access;
- model quality and rollback;
- security and privacy incidents;
- user reports and moderation;
- user support and field feedback.

Do not launch a feature whose critical operational responsibility is unassigned.
