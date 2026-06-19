# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: End Tests Validating

These rules are ALWAYS ACTIVE for all public API contracts, state management reducers, browser-based APIs, and E2E utility functions that require validation across unit and integration boundaries.

### Rules

- **R-API-001** MUST: End-to-end tests validating browser-based API contracts MUST use puppeteer with page.evaluate, page.goto, and browser.newPage to verify runtime behavior in actual browser contexts.
- **R-API-002** MUST: All exported public API contracts including configuration objects, data structures, and utility functions MUST have corresponding unit tests using jest.mock for dependency isolation.
- **R-API-003** MUST: State management reducers and actions that transform public data structures MUST be validated using expect().toEqual() for state objects, indexes, and transformed data representations.
- **R-API-004** MUST: Browser-based APIs that execute in page contexts or handle DOM interactions MUST be tested with E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts.
- **R-API-005** SHOULD: E2E utility functions that coordinate browser automation workflows SHOULD be exported and reused across test suites to maintain consistent browser automation patterns.
- **R-API-006** SHOULD: Complex page.evaluate() logic for iframe transformations SHOULD be encapsulated in well-documented utility functions with corresponding unit tests for transformation calculations where possible.

### Verify

```bash
# Verify Jest mocking for dependency isolation
grep -r "jest.mock" packages/core/reducer/actions/__helpers__/ | grep -q "generate-id" && echo "Jest mocking verified"

# Verify Puppeteer E2E setup
grep -r "puppeteer.launch" scripts/e2e/utils/ | grep -q "headless" && echo "Puppeteer E2E setup verified"

# Verify state contract validation
grep -r "expect.*toEqual" packages/core/reducer/actions/__helpers__/ | grep -q "state.indexes" && echo "State contract validation verified"

# Verify browser context evaluation
grep -r "page.evaluate" scripts/e2e/utils/ | grep -q "querySelector" && echo "Browser context evaluation verified"
```

**Accept when:**
- All public API contracts have corresponding unit tests using jest.mock for dependency isolation and expect().toEqual() for validation
- Browser-based APIs have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase
- New public API contracts include tests at appropriate architectural boundaries (unit for logic, E2E for browser behavior)

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API contracts and browser-based APIs MUST be tested according to R-API-001 through R-API-006. Pull requests without appropriate test coverage for new public API contracts are blocked from merging. CI failures in unit or E2E test suites prevent deployment.
</enforcement>