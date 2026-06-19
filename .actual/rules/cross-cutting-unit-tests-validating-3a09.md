# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: Unit Tests Validating

These rules are ALWAYS ACTIVE for all files matching the configured scope: exported public API contracts, state management reducers and actions, browser-based APIs, and E2E utility functions.

### Rules

- **R-API-001** MUST: Unit tests validating state transformations and data structures MUST use jest.mock to isolate external dependencies and ensure deterministic test execution.
- **R-API-002** MUST: All exported public API contracts including configuration objects, data structures, and utility functions MUST have corresponding test coverage at appropriate architectural boundaries.
- **R-API-003** MUST: Browser-based APIs that execute in page contexts or handle DOM interactions MUST be validated with E2E tests using puppeteer and page.evaluate().
- **R-API-004** SHOULD: State validation in unit tests SHOULD use expect().toEqual() to verify both shape and content of data structures including state objects and indexes.
- **R-API-005** SHOULD: E2E utility functions like setup and getBox SHOULD be exported to enable reuse across test suites and maintain consistent browser automation patterns.

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
- Browser-based APIs have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase
- Pull requests include test coverage for new public API contracts at appropriate boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline executes both Jest unit tests and Puppeteer E2E tests on every pull request. Pull requests without tests for new public API contracts are blocked from merging. Code review checklist requires test coverage verification before approval.
</enforcement>