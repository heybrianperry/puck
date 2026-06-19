# Adopt Puppeteer for End-to-End Browser Testing: Test Scripts Use

These rules are ALWAYS ACTIVE for all end-to-end test scripts in the `scripts/e2e/` directory and smoke test frameworks that validate application behavior in browser environments.

### Rules

- **R-E2E-001** MUST: Use Puppeteer for browser automation in E2E test scripts, following the established pattern: `const browser = await puppeteer.launch({ headless, ...options }); const page = await browser.newPage(); await page.goto(url);`
- **R-E2E-002** MUST: Leverage existing setup utilities in `scripts/e2e/utils/setup.mjs` rather than duplicating browser initialization logic.
- **R-E2E-003** MUST: Ensure proper cleanup by closing pages and browsers in finally blocks to prevent resource leaks during test execution.
- **R-E2E-004** SHOULD: Implement retry logic, explicit waits for elements, and network idle detection to reduce test flakiness.
- **R-E2E-005** SHOULD: Wrap performance.memory access in try-catch blocks and gracefully degrade when APIs are unavailable.
- **R-E2E-006** MAY: Test scripts MAY use asciichart or similar libraries for visualizing performance metrics during test runs.
- **R-E2E-007** SHOULD: When implementing smoke tests, sample `performance.memory.usedJSHeapSize` at intervals and compare against thresholds to detect memory leaks.

### Verify

```bash
# Verify Puppeteer usage in E2E test scripts
grep -r "puppeteer.launch" scripts/e2e/

# Verify standard Puppeteer API patterns
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring integration
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in `scripts/e2e/` directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options
- Smoke tests monitor performance metrics during execution and log results
- Browser instances are properly closed in finally blocks
- Performance monitoring APIs are wrapped in error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All new E2E test scripts must pass grep verification and code review checks for Puppeteer usage patterns before merge.
</enforcement>