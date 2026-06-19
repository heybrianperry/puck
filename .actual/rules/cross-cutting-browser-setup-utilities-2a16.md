# Adopt Puppeteer for End-to-End Browser Testing: Browser Setup Utilities

These rules are ALWAYS ACTIVE for all end-to-end test scripts and browser automation utilities in the `scripts/e2e/` directory and related smoke test frameworks.

### Rules

- **R-PUPPETEER-001** MUST: Browser setup utilities MUST expose configuration options for headless mode and other launch parameters.

### Verify

```bash
# Verify Puppeteer is used for browser automation
grep -r "puppeteer.launch" scripts/e2e/

# Verify standard Puppeteer API patterns are in use
grep -r "browser.newPage\|page.goto" scripts/e2e/

# Verify performance monitoring is integrated in smoke tests
grep -r "performance.memory.usedJSHeapSize" scripts/e2e/
```

**Accept when:**
- All E2E test scripts in `scripts/e2e/` directory use Puppeteer API for browser automation
- Setup utilities expose reusable functions for browser initialization with configurable options (headless mode, launch parameters)
- Smoke tests monitor performance metrics during execution and log results
- Browser instances are properly initialized using `puppeteer.launch({ headless, ...options })` pattern
- Pages are created and navigated using `browser.newPage()` and `page.goto(url)` patterns
- Resource cleanup occurs in finally blocks to prevent leaks

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All E2E test files must be checked against the Puppeteer API patterns and configuration requirements before approval.
</enforcement>