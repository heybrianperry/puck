# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: Public Contracts Userconfig

These rules are ALWAYS ACTIVE for all public API contracts, state management reducers, actions, and browser-based APIs that require validation across unit and integration boundaries.

### Rules

- **R-CONTRACT-001** MUST: Public API contracts (UserConfig, UserData, setup, getBox, etc.) MUST be validated through automated tests at both unit and integration boundaries.
- **R-CONTRACT-002** MUST: Unit tests for public API contracts MUST use jest.mock() to isolate dependencies and expect().toEqual() to validate state transformations and data structure contracts.
- **R-CONTRACT-003** MUST: Browser-based APIs and E2E scenarios MUST be validated using Puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts.
- **R-CONTRACT-004** MUST: All exported public API contracts including configuration objects, data structures, and utility functions MUST have corresponding test coverage.
- **R-CONTRACT-005** SHOULD: E2E tests SHOULD implement retry logic, explicit waits, and stable selectors to reduce flakiness from timing issues and browser version changes.
- **R-CONTRACT-006** SHOULD: Complex page.evaluate() logic for iframe transformations SHOULD be encapsulated in well-documented utility functions with unit tests for transformation calculations where possible.

### Verify

```bash
# Verify jest.mock usage for dependency isolation
grep -r "jest.mock" packages/core/reducer/actions/__helpers__/ | grep -q "generate-id" && echo "Jest mocking verified"

# Verify Puppeteer E2E setup
grep -r "puppeteer.launch" scripts/e2e/utils/ | grep -q "headless" && echo "Puppeteer E2E setup verified"

# Verify state contract validation patterns
grep -r "expect.*toEqual" packages/core/reducer/actions/__helpers__/ | grep -q "state.indexes" && echo "State contract validation verified"

# Verify browser context evaluation
grep -r "page.evaluate" scripts/e2e/utils/ | grep -q "querySelector" && echo "Browser context evaluation verified"
```

**Accept when:**
- All public API contracts have corresponding unit tests using jest.mock for dependency isolation and expect().toEqual() for validation
- Browser-based APIs have E2E tests using Puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, Puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase
- New public API contracts include test coverage at appropriate architectural boundaries before merging

<enforcement>
Claude Code MUST NOT skip or defer verification of public API contract testing requirements. All pull requests introducing new public APIs or modifying existing contracts MUST include both unit and E2E test coverage as appropriate. CI pipeline execution of Jest and Puppeteer tests is mandatory on every pull request.
</enforcement>