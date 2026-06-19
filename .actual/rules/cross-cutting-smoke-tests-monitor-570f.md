# Adopt Puppeteer for End-to-End Browser Testing: Smoke Tests Monitor

These rules are ALWAYS ACTIVE for all end-to-end browser automation tests and smoke test frameworks in the `scripts/e2e/` directory.

### Rules

- **R-E2E-001** SHOULD: Smoke tests SHOULD monitor performance metrics including memory usage (performance.memory.usedJSHeapSize) during test execution to detect resource leaks and validate application performance under automated testing conditions.

- **R-E2E-002** MUST: All E2E test scripts MUST use Puppeteer for browser automation, following the established pattern: `const browser = await puppeteer.launch({ headless, ...options }); const page = await browser.newPage(); await page.goto(url);`

- **R-E2E-003** SHOULD: E2E test setup SHOULD leverage existing reusable utilities in `scripts/e2e/utils/setup.mjs` rather than duplicating browser initialization logic.

- **R-E2E-004** MUST: Browser instances and pages MUST be properly closed in finally blocks to prevent resource leaks during test execution.

- **R-E2E-005** SHOULD: Performance monitoring in smoke tests SHOULD sample `performance.memory.usedJSHeapSize` at intervals and compare against thresholds to detect memory leaks.

- **R-E2E-006** SHOULD: Performance monitoring API access SHOULD be wrapped in try-catch blocks to gracefully degrade when APIs are unavailable in different execution contexts.

### Verify

```bash
# Verify Puppeteer is used for browser automation
grep -r "puppeteer.launch" scripts/e2e/

# Verify standard Puppeteer API patterns are present
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring is implemented in smoke tests
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in `scripts/e2e/` directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options
- Smoke tests monitor performance metrics during execution and log results
- Browser instances are properly closed in finally blocks or cleanup handlers
- Performance monitoring access is wrapped in error handling for graceful degradation

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new E2E test scripts must conform to the Puppeteer adoption pattern and performance monitoring requirements before merge.
</enforcement>