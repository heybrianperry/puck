# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: Tests Validating Public

These rules are ALWAYS ACTIVE for all files exporting public API contracts, state management reducers and actions, browser-based APIs, and E2E utility functions that require validation across unit and integration boundaries.

### Rules

- **R-API-001** MUST: Tests validating public API contracts MUST verify both data structure correctness (using expect().toEqual()) and path/index consistency in state management.
- **R-API-002** MUST: All exported public API contracts including configuration objects, data structures, and utility functions MUST have corresponding test coverage at appropriate architectural boundaries.
- **R-API-003** MUST: Unit tests validating state management MUST use jest.mock() to isolate dependencies and ensure deterministic test execution.
- **R-API-004** MUST: Browser-based APIs MUST have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts.
- **R-API-005** SHOULD: E2E tests SHOULD implement retry logic, explicit waits, and stable selectors to minimize flakiness due to timing issues or browser version changes.
- **R-API-006** SHOULD: Complex page.evaluate() logic for iframe transformations SHOULD be encapsulated in well-documented utility functions with corresponding unit tests for transformation calculations.

### Verify

```bash
# Verify Jest mocking for dependency isolation
grep -r "jest.mock" packages/core/reducer/actions/__helpers__/ | grep -q "generate-id" && echo "Jest mocking verified"

# Verify Puppeteer E2E setup
grep -r "puppeteer.launch" scripts/e2e/utils/ | grep -q "headless" && echo "Puppeteer E2E setup verified"

# Verify state contract validation with expect().toEqual()
grep -r "expect.*toEqual" packages/core/reducer/actions/__helpers__/ | grep -q "state.indexes" && echo "State contract validation verified"

# Verify browser context evaluation
grep -r "page.evaluate" scripts/e2e/utils/ | grep -q "querySelector" && echo "Browser context evaluation verified"
```

**Accept when:**
- All public API contracts have corresponding unit tests using jest.mock for dependency isolation and expect().toEqual() for validation
- Browser-based APIs have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase
- New public API contracts include tests at appropriate architectural boundaries before merging

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Pull requests without tests for new public API contracts MUST be blocked from merging. CI failures in unit or E2E test suites MUST prevent deployment to staging and production environments.
</enforcement>