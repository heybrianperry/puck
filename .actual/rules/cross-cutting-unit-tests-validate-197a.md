# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: Unit Tests Validate

These rules are ALWAYS ACTIVE for all public API contracts, state management reducers, actions, and browser-based APIs that require validation across unit and integration boundaries.

### Rules

- **R-API-001** SHOULD: Unit tests SHOULD validate flattened data representations alongside original data structures to ensure transformation consistency.
- **R-API-002** SHOULD: Use jest.mock() at the top of test files to isolate dependencies like generate-id, ensuring mocks are hoisted before module imports.
- **R-API-003** SHOULD: Structure E2E tests with clear setup/teardown: launch browser with appropriate options, create new pages, navigate to URLs, and clean up resources.
- **R-API-004** SHOULD: When testing APIs that interact with iframes, use page.evaluate() to execute transformation logic in the browser context where DOM APIs are available.
- **R-API-005** SHOULD: Validate both the shape and content of data structures using expect().toEqual() for state objects, indexes, and transformed data representations.
- **R-API-006** SHOULD: Export E2E utility functions like setup and getBox to enable reuse across test suites and maintain consistent browser automation patterns.

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
- All exported public API contracts including configuration objects, data structures, and utility functions are tested at appropriate architectural boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API contracts require validation at multiple architectural boundaries. CI pipeline executes both Jest unit tests and Puppeteer E2E tests on every pull request. Code review checklist requires test coverage for new public API contracts. Pull requests without tests for new public API contracts are blocked from merging.
</enforcement>