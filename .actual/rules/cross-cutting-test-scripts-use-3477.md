# Adopt Puppeteer for End-to-End Browser Testing: Test Scripts Use

These rules are ALWAYS ACTIVE for all end-to-end test scripts and smoke test frameworks in the scripts/e2e/ directory that require programmatic browser automation.

### Rules

- **R-E2E-001** MUST: Test scripts MUST use the standard Puppeteer API pattern: `puppeteer.launch()` for browser instances, `browser.newPage()` for page creation, and `page.goto()` for navigation.
- **R-E2E-002** MUST: Browser initialization MUST use existing setup utilities in scripts/e2e/utils/setup.mjs rather than duplicating browser initialization logic.
- **R-E2E-003** MUST: Smoke test frameworks MUST monitor performance metrics including `performance.memory.usedJSHeapSize` at intervals to detect memory leaks.
- **R-E2E-004** MUST: Browser and page resources MUST be properly closed in finally blocks to prevent resource leaks during test execution.
- **R-E2E-005** SHOULD: Performance monitoring API access SHOULD be wrapped in try-catch blocks to gracefully degrade when APIs are unavailable.

### Verify

```bash
# Verify Puppeteer launch pattern usage
grep -r "puppeteer.launch" scripts/e2e/

# Verify page creation and navigation patterns
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring integration
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/

# Verify setup utility usage in test files
grep -r "from.*setup.mjs\|require.*setup.mjs" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in scripts/e2e/ directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options
- Smoke tests monitor performance metrics during execution and log results
- Browser and page cleanup is present in test teardown or finally blocks

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing or generating E2E test code.
</enforcement>