# 01 — Project Charter

## Executive summary

**MandiMitra** is a proposed agricultural market intelligence and direct-trade platform for Indian farmers and produce buyers. It combines market-price discovery, short-horizon price estimation, multilingual voice-assisted workflows, produce listings, competitive buyer offers, and authenticated conversations.

This charter describes a project blueprint. It does not claim that the application, integrations, or forecasting model already exist.

## Vision

Make agricultural market information easier to understand and make produce discovery and offer comparison more transparent, accessible, and accountable.

## Mission

- Present observed mandi prices with source, date, unit and market context.
- Produce cautious, explainable price estimates only when data is sufficient.
- Support voice and local-language access without making voice mandatory.
- Let farmers publish produce lots and receive structured buyer offers.
- Help farmers compare offers using quantity, price, quality terms, location and estimated logistics cost.
- Create safer communication between authenticated marketplace participants.

## Problem statement

Information about agricultural prices and selling opportunities can be fragmented across markets and channels. A headline price does not automatically represent the farmer's net realization: distance, loading, quality, grading, commissions, buyer terms and timing can affect the final outcome. Digital interfaces can also be difficult for users unfamiliar with complex forms or English-first products.

MandiMitra brings relevant information and marketplace actions into a single workflow. It will not claim to eliminate intermediaries, control market prices, guarantee a sale, or guarantee increased income.

## Objectives and measures

| Objective | Measurement | Evaluation |
|---|---|---|
| Improve market information access | Price coverage and freshness | Pilot market/commodity combinations with recent, valid observations |
| Provide useful price outlooks | Forecast error and interval coverage | Rolling-origin backtesting against naive baselines |
| Reduce workflow friction | Task completion and time-on-task | Moderated usability sessions in each supported language |
| Enable buyer discovery | Qualified buyer response rate | Listings receiving a relevant enquiry or bid |
| Improve offer clarity | Offer-comparison completion | Farmer sessions comparing at least two complete offers |
| Operate responsibly | Safety and support measures | Reports, unresolved disputes and support response time |

Targets will be agreed after discovery and baseline measurement. No unsupported numerical outcome is assumed.

## Stakeholders

| Stakeholder | Interest / responsibility |
|---|---|
| Farmers / sellers | Market discovery, produce listings, offers and sale decisions |
| Buyers | Find produce, review lots, submit offers and coordinate fulfilment |
| Field partners | Onboarding support, local terminology and pilot feedback |
| Product owner | Scope, priorities, acceptance and release decisions |
| Engineering team | Mobile, web, backend, data, ML and reliability |
| Data steward | Source verification, schema mapping and data quality |
| Support / operations | Reports, moderation, incidents and user support |
| External data providers | Source data, usage terms, availability and attribution |

## Scope boundaries

### Included in the planned product

- Farmer, buyer and administrator roles.
- Initial multilingual support, proposed as English, Telugu and Hindi.
- Market and commodity search.
- Observed minimum, maximum and modal prices where the source provides them.
- Historical price charts and market comparisons.
- Forecast ranges with dates, horizons, uncertainty and freshness.
- Farmer produce lots with quantity, unit, variety, quality notes, images and location.
- Buyer discovery and structured bids.
- Offer comparison, acceptance and rejection states.
- Authenticated listing-linked conversations.
- Notifications, reporting, audit events and administration.

### Excluded from the first MVP

- Guaranteed prices, guaranteed buyers or guaranteed farmer income.
- Automatic sale execution without farmer confirmation.
- Escrow, lending, credit scoring, insurance or financial products.
- Owning or guaranteeing transport, warehousing, weighing or certification.
- Replacing official mandi systems or government platforms.
- National coverage at launch.
- A claim of end-to-end encrypted messaging without a separately verified design.

## Assumptions

1. The pilot covers a limited number of districts, markets and commodities.
2. Market-data access and terms can be validated before forecasting is released.
3. Farmers may use low-cost Android devices and inconsistent connectivity.
4. Voice capability depends on device and language-service availability; text remains available.
5. Buyers can provide structured quantities, prices and terms.
6. Human support and moderation will exist for the pilot.
7. Any verification process will be proportionate and explained to participants.

## Constraints

- Market observations may be late, missing, or use inconsistent names and units.
- Price behavior differs by commodity, variety, grade, region, season and market.
- Forecast quality may be weak for sparse or newly listed commodities.
- Speech recognition varies by accent, noise, device and language.
- Marketplace trust requires field validation, not only software testing.
- Privacy, retention and consumer-protection obligations require review for the actual launch model.
- External provider terms and rate limits may change.

## Risk register

| Risk | Impact | Mitigation |
|---|---|---|
| Stale or incomplete data | Misleading decisions | Show source timestamp, validate rows, flag stale feeds and suppress unsupported claims |
| Weak forecast performance | False confidence | Compare with baselines, publish interval coverage and abstain when data is insufficient |
| Unit or variety mismatch | Invalid comparisons | Normalize units, retain source labels and reject incompatible comparisons |
| Low buyer participation | Few offers | Pilot with a bounded buyer network and track liquidity by crop and district |
| Fraudulent listings or bids | Trust harm | Reporting, rate limits, moderation, audit history and explicit offer states |
| Speech transcription errors | Incorrect action | Confirm extracted values before submission; allow edit and cancel |
| Network interruptions | Lost work | Draft persistence, bounded retries and visible sync state |
| Privacy leakage | Personal data exposure | Least privilege, private storage, RLS/API authorization and restricted contact sharing |

## Project success

The pilot is successful when representative farmers and buyers can complete the key workflows safely and reliably; the data pipeline has measurable coverage and freshness; forecast performance is transparently evaluated against baselines; and the team can support reports, failures and corrections. Adoption or income claims require separate evidence from a properly designed field evaluation.

## Governance

- Product owner approves scope changes and release gates.
- Each requirement has an identifier and acceptance criteria.
- Architecture and data-source decisions are recorded as ADRs.
- Data lineage, model version and release version are retained.
- Material incidents and model regressions are documented.
- User-facing claims must match measured capability.

## Decisions required before implementation

| Decision | Evidence / owner |
|---|---|
| Pilot geography and initial crops | Farmer/buyer discovery and source coverage / Product |
| Data source route and terms | Verified dataset, API documentation and provider terms / Data |
| Initial language set | Pilot research and device speech support / Product + UX |
| Buyer onboarding and verification | Operational feasibility and misuse risk / Operations |
| Logistics-cost estimation | Local rate availability and validation / Product + Data |
| Payments in a later phase | Legal, operational and partner review / Product |
| Hosting region and retention | Privacy review and provider capabilities / Engineering |
