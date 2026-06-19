# Adopt Puppeteer for End-to-End Browser Testing: E2e Test Utilities

These rules are ALWAYS ACTIVE for all end-to-end test scripts and utilities in the `scripts/e2e/` directory that perform browser automation and smoke testing.

### Rules

- **R-E2E-001** SHOULD: E2E test utilities SHOULD be organized in `scripts/e2e/utils/` directory with reusable setup functions that encapsulate browser initialization with configurable options including headless mode.
- **R-E2E-002** MUST: All E2E test scripts in `scripts/e2e/` directory MUST use Puppeteer API for browser automation following the established pattern: `const browser = await puppeteer.launch({ headless, ...options }); const page = await browser.newPage(); await page.goto(url);`
- **R-E2E-003** SHOULD: E2E test utilities SHOULD leverage existing setup utilities in `scripts/e2e/utils/setup.mjs` rather than duplicating browser initialization logic.
- **R-E2E-004** SHOULD: Smoke tests SHOULD monitor performance metrics during execution by sampling `performance.memory.usedJSHeapSize` at intervals and comparing against thresholds to detect memory leaks.
- **R-E2E-005** MUST: Browser automation tests MUST ensure proper cleanup by closing pages and browsers in finally blocks to prevent resource leaks during test execution.
- **R-E2E-006** SHOULD: Performance monitoring API access SHOULD be wrapped in try-catch blocks to gracefully degrade when APIs are unavailable in different execution contexts.

### Verify

```bash
# Verify Puppeteer usage in E2E test scripts
grep -r "puppeteer.launch" scripts/e2e/

# Verify standard Puppeteer API patterns
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring in smoke tests
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in `scripts/e2e/` directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options
- Smoke tests monitor performance metrics during execution and log results
- Browser cleanup is implemented in finally blocks or equivalent error-safe patterns
- Performance monitoring API calls are protected with error handling

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing or modifying E2E test infrastructure. All R-E2E rules must be validated against the verify commands before accepting changes to scripts/e2e/ directory.
</enforcement>