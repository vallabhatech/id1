# Contributing to MandiMitra

Thanks for helping improve MandiMitra. The project is currently a planning repository; implementation contributions should follow the requirements, architecture and safety rules documented in the docs folder.

## 1. Before contributing

1. Read the project overview in README.md.
2. Review the project charter and product requirements.
3. Check the implementation roadmap and planned repository structure.
4. Open or discuss an issue for substantial changes before starting work.
5. Confirm whether the change affects privacy, authorization, market-data provenance, forecast claims or marketplace state.

Do not treat a proposed design in the docs as an already implemented feature.

## 2. Contribution principles

- Keep farmer tasks simple and understandable.
- Preserve the distinction between source observations and model estimates.
- Never invent market data, units, buyer verification or forecast accuracy.
- Keep user information private by default.
- Validate permissions on the server and database, not only in the UI.
- Prefer small, reviewable changes over large unrelated commits.
- Add tests for changed behavior.
- Update documentation when behavior, API contracts or data models change.

## 3. Branch and pull-request workflow

Suggested branch naming:
- feature/short-description
- fix/short-description
- docs/short-description
- test/short-description
- chore/short-description

Workflow:
1. Create a branch from the latest main.
2. Make a focused change.
3. Run relevant formatters, linters and tests.
4. Update documentation and examples.
5. Open a pull request with context, screenshots where useful, test evidence and risks.
6. Address review comments and merge only after required checks pass.

Do not commit credentials, private user data, production exports, unapproved datasets or generated model artifacts.

## 4. Commit message convention

Use a simple conventional prefix:

| Prefix | Use |
|---|---|
| feat | New user-facing capability |
| fix | Bug fix |
| docs | Documentation-only change |
| test | Test additions or corrections |
| refactor | Behavior-preserving code restructuring |
| chore | Tooling, dependencies or maintenance |
| security | Security/privacy improvement |

Examples:
- docs: clarify price freshness behavior
- feat: add farmer mandi search
- fix: prevent duplicate bid acceptance
- test: cover market unit normalization

## 5. Code quality expectations

### General
- Use clear names and small functions.
- Keep business rules out of presentation components.
- Validate input at system boundaries.
- Handle loading, empty, offline and failure states.
- Avoid logging personal data, message bodies, tokens or exact coordinates.
- Use UTC for stored timestamps.
- Use decimal-safe types for prices and quantities.

### Android
- Keep UI state predictable and lifecycle-aware.
- Use accessible labels and scalable text.
- Provide manual alternatives to voice flows.
- Do not block essential workflows on microphone permission.
- Make offline draft and sync state visible.

### Web
- Use semantic HTML and accessible form controls.
- Enforce server-side authorization for protected pages and actions.
- Avoid exposing secrets in client bundles.
- Keep user-submitted content safely rendered.

### Python API and data services
- Use typed request and response models.
- Keep provider parsing inside adapters.
- Make ingestion idempotent and record run metadata.
- Separate training/evaluation code from request-time inference.
- Add time-aware ML tests to guard against data leakage.

## 6. Testing expectations

For each contribution:
- Add or update unit tests for business rules.
- Add API/integration tests for changed endpoints.
- Test authorization and ownership where data access changes.
- Test error, empty and stale-data states.
- Test source mapping and units for market-data changes.
- Test transactional behavior for listing and bid state changes.
- Include model evaluation evidence for forecasting changes.

The test strategy is documented in docs/09-testing-and-quality.md.

## 7. Documentation expectations

Documentation must identify whether a feature is:
- planned,
- in progress,
- implemented,
- tested,
- deployed,
- or disabled.

Do not label a feature as live until it has been implemented and verified. Do not invent screenshots or demo URLs. Use synthetic data in examples and label it clearly.

## 8. Pull-request checklist

- [ ] Scope is focused and linked to an issue or requirement.
- [ ] Relevant tests pass.
- [ ] Authorization/privacy impact has been considered.
- [ ] No secret or personal data is committed.
- [ ] UI supports accessibility and the supported locale behavior.
- [ ] Data source, unit and freshness semantics remain correct.
- [ ] API/database documentation is updated where needed.
- [ ] Screenshots or recordings are synthetic/demo-safe where applicable.
- [ ] Limitations and rollout risks are stated.

## 9. Reporting security issues

Do not publish credentials, private farmer information or exploitable details in a public issue. Use the repository owner's private security reporting channel where configured. Until a dedicated channel is published, avoid including sensitive details in public discussions and contact the maintainer through an appropriate private route.
