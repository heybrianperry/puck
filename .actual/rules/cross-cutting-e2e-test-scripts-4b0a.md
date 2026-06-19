# Adopt Puppeteer for End-to-End Browser Testing: E2e Test Scripts

These rules are ALWAYS ACTIVE for all end-to-end test scripts and smoke testing frameworks in the `scripts/e2e/` directory that require programmatic browser control and automation.

### Rules

- **R-E2E-001** MUST: E2E test scripts MUST use Puppeteer as the browser automation library for launching browsers and controlling pages.
- **R-E2E-002** MUST: Browser initialization MUST follow the established pattern: `const browser = await puppeteer.launch({ headless, ...options }); const page = await browser.newPage(); await page.goto(url);`
- **R-E2E-003** MUST: E2E test scripts MUST leverage existing setup utilities in `scripts/e2e/utils/setup.mjs` rather than duplicating browser initialization logic.
- **R-E2E-004** MUST: Proper cleanup MUST be ensured by closing pages and browsers in finally blocks to prevent resource leaks during test execution.
- **R-E2E-005** SHOULD: Smoke tests SHOULD sample `performance.memory.usedJSHeapSize` at intervals and compare against thresholds to detect memory leaks.
- **R-E2E-006** SHOULD: Performance monitoring API access SHOULD be wrapped in try-catch blocks to gracefully degrade when APIs are unavailable.

### Verify

```bash
# Verify Puppeteer launch pattern usage
grep -r "puppeteer.launch" scripts/e2e/

# Verify browser and page API usage
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring integration
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in `scripts/e2e/` directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options
- Smoke tests monitor performance metrics during execution and log results
- Browser and page instances are properly closed in finally blocks
- Performance monitoring access is wrapped in error handling where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep patterns MUST pass before accepting changes to E2E test infrastructure. Pull requests introducing alternative browser automation libraries without documented justification MUST be rejected.
</enforcement>