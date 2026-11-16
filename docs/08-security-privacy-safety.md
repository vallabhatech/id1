# 08 — Security, Privacy and Marketplace Safety

## 1. Security objectives

MandiMitra will handle accounts, location context, listings, offers and private conversations. Security must be designed into the system before a public pilot.

Objectives:
- Protect account access and private user information.
- Prevent one user from reading or modifying another user's resources.
- Keep privileged secrets out of client applications and repositories.
- Make marketplace state changes traceable.
- Limit spam, impersonation, fraudulent listings and abusive messages.
- Give users clear control over their profile and communication preferences.

## 2. Data classification

| Classification | Examples | Handling |
|---|---|---|
| Public | Published listing fields, canonical market references, public market prices | Public read only after review and policy checks |
| Internal | Ingestion metadata, system metrics, reference mapping notes | Staff/service access only |
| Personal | Display name, district, account identifiers, language preference | Restricted profile access; minimize collection |
| Sensitive operational | Exact pickup location, contact details, private images, chat content, reports | Strict authorization, retention policy and encrypted transport/storage |
| Secret | Access tokens, service keys, signing secrets, provider credentials | Secret manager/environment only; never client-side or committed |

## 3. Authentication and authorization

- Use a maintained identity provider for sign-in and account recovery.
- Verify access tokens on every API request.
- Check token signature, issuer, audience and expiry.
- Do not treat a user-selected role in a request body as an authorization grant.
- Enforce ownership and role permissions in the backend.
- Enable database row-level security for tables accessed through client-facing services.
- Use a separate privileged backend identity only for tasks that require it.
- Require additional protection for administrator accounts and privileged operations.
- Revoke sessions and device tokens when account status requires it.

### Authorization rule

A request is allowed only when the authenticated actor, resource ownership, participant relationship, resource state and operation are all compatible. Hiding a button in the UI is not access control.

## 4. Secret management

- Store API keys, database service-role keys, signing secrets and provider credentials in the hosting platform's secret manager.
- Use separate credentials for development, staging and production.
- Never include secrets in Android resources, web bundles, logs, screenshots or public repositories.
- Rotate credentials after suspected exposure and on a defined operational schedule.
- Restrict provider keys to required APIs and origins/IPs where provider support allows it.
- Scan commits and dependencies for accidentally committed secrets.

Public client configuration may contain only keys explicitly designed by the provider to be public and protected by server-side/database policies.

## 5. Privacy principles

- Collect the minimum profile information needed for the feature.
- Explain why location is requested before requesting permission.
- Prefer district/market-level public location over exact coordinates.
- Keep exact pickup location private until the seller agrees to share it with an eligible buyer.
- Do not publish phone numbers or private contact information by default.
- Do not use chat content or precise location for unrelated analytics.
- Provide a clear privacy notice, contact path and retention explanation.
- Define account export/deletion processes consistent with applicable requirements.
- Use third-party speech services only after reviewing their permission, data handling and retention behavior; offer text fallback.

## 6. Marketplace safety controls

| Threat | Planned control |
|---|---|
| Fake or misleading listing | Report workflow, suspicious-content review, listing state controls and audit trail |
| Buyer impersonation | Account authentication, proportionate verification status and user education |
| Bid manipulation | Bid history, explicit validity, state transition rules and rate limits |
| Multiple acceptances | Transactional bid acceptance and listing-level concurrency control |
| Harassment or spam | Block/report controls, throttling and moderation workflow |
| Contact scraping | Restricted profile projections, private contact fields and API rate limits |
| Malicious uploads | File type/size validation, private storage, safe processing and controlled download |
| Account takeover | Identity-provider protections, recovery controls, session revocation and alerting |
| Admin misuse | Least privilege, audited actions and restricted investigation access |

Verification labels must describe the exact verification performed. A phone or email check must not be displayed as proof of legal identity or business legitimacy.

## 7. Messaging privacy

- Messages are visible only to authorized conversation participants and approved support investigators under a documented process.
- Access is checked server-side for each message read/write and realtime subscription.
- Do not claim end-to-end encryption unless the design, key management and implementation have been independently reviewed.
- Define message retention, deletion, reporting and legal-hold behavior before pilot.
- Never include message bodies in routine analytics or standard application logs.
- Provide a report action that links the reported message to the appropriate moderation case without exposing it publicly.

## 8. API and application security

- HTTPS for all service traffic.
- Strict request validation and output projection.
- Rate limits for sign-in-adjacent endpoints, search, chat, reports and media upload.
- CORS allowlist for trusted web origins.
- CSRF protection where cookie-based authentication is used.
- Secure cookie settings where applicable.
- Content Security Policy for web application surfaces.
- Avoid unsafe HTML rendering of user-submitted content.
- Validate uploaded file signatures in addition to file extensions.
- Use dependency scanning and update vulnerable packages.
- Keep framework and runtime versions supported.
- Return generic errors to users and detailed sanitized errors to restricted logs.

## 9. Audit requirements

Record at least:
- Listing publish, pause, close and moderation decisions.
- Bid submit, revise, withdraw, accept and decline state changes.
- Account role or verification changes.
- Report assignment and resolution.
- Reference-data changes that affect price mapping.
- Privileged data export or administrative access where required.

Audit events should include actor, action, target, timestamp, request/correlation ID and a reason where appropriate. Do not store secrets or unnecessary message bodies in audit metadata. Regular users must not be able to edit or delete audit events.

## 10. Abuse prevention and support

Before pilot, define:
- Report categories and severity.
- Who receives each report type.
- Response expectations and escalation route.
- Criteria for hiding a listing or suspending an account.
- Appeal or correction process.
- Emergency contact and incident owner.
- How users are informed when an action affects their account or listing.

Human moderation is required for high-impact or ambiguous cases. Automated flags should support review, not silently make irreversible decisions.

## 11. Incident response

1. Detect and classify the incident.
2. Contain affected credentials, sessions, endpoints or records.
3. Preserve relevant logs and audit evidence.
4. Assess affected data and users.
5. Notify responsible stakeholders and users/regulators where required by applicable rules.
6. Recover service and rotate credentials as necessary.
7. Document root cause, impact, remediation and prevention work.

## 12. Pre-pilot security review

- [ ] Threat model reviewed for farmer, buyer, admin and external provider boundaries.
- [ ] RLS and API authorization tested using separate accounts for all roles.
- [ ] No privileged keys in client bundles or Git history.
- [ ] Private files cannot be fetched by unauthorized users.
- [ ] Conversation membership enforced for HTTP and realtime access.
- [ ] Bid acceptance race conditions tested.
- [ ] Rate limits and upload constraints enabled.
- [ ] Session revocation and account recovery tested.
- [ ] Privacy notice and retention rules approved.
- [ ] Backup restore and incident owner established.
