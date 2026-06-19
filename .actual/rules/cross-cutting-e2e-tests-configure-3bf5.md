# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: E2e Tests Configure

These rules are ALWAYS ACTIVE for all files matching the configured scope: public API contracts, state management reducers and actions, browser-based APIs, and E2E utility functions that coordinate browser automation workflows.

### Rules

- **R-E2E-001** MAY: E2E tests MAY configure puppeteer with headless mode options to support both CI and local debugging workflows.

### Verify

```bash
# Verify Jest mocking for dependency isolation
grep -r "jest.mock" packages/core/reducer/actions/__helpers__/ | grep -q "generate-id" && echo "Jest mocking verified"

# Verify Puppeteer E2E setup with headless configuration
grep -r "puppeteer.launch" scripts/e2e/utils/ | grep -q "headless" && echo "Puppeteer E2E setup verified"

# Verify state contract validation using expect().toEqual()
grep -r "expect.*toEqual" packages/core/reducer/actions/__helpers__/ | grep -q "state.indexes" && echo "State contract validation verified"

# Verify browser context evaluation with page.evaluate()
grep -r "page.evaluate" scripts/e2e/utils/ | grep -q "querySelector" && echo "Browser context evaluation verified"
```

**Accept when:**
- All public API contracts have corresponding unit tests using jest.mock for dependency isolation and expect().toEqual() for validation
- Browser-based APIs have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase
- E2E tests configure puppeteer with headless mode options that support both CI and local debugging workflows

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API contracts must be validated at multiple architectural boundaries using the dual-layer testing strategy (Jest mocking for unit tests, Puppeteer for E2E tests). CI pipeline execution of both test suites on every pull request is mandatory.
</enforcement>