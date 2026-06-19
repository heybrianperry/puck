# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: Page Navigation Use

These rules are ALWAYS ACTIVE for end-to-end tests validating public API contracts through browser automation, specifically tests in the `scripts/e2e/` directory that involve page navigation and element inspection across iframe boundaries.

### Rules

- **R-PUPPETEER-001** MUST: Page navigation MUST use `page.goto(url)` for loading test targets.

### Verify

```bash
# Verify Puppeteer launch configuration with headless option support
grep -r 'puppeteer.launch' scripts/e2e/ | grep -q 'headless'

# Verify frame transformation logic exists in page.evaluate calls
grep -r 'page.evaluate' scripts/e2e/ | grep -q 'getFrameTransform'

# Verify both setup and get-box utility files exist
test -f scripts/e2e/utils/setup.mjs && test -f scripts/e2e/utils/get-box.mjs
```

**Accept when:**
- Puppeteer launch configuration is present in `scripts/e2e/utils/setup.mjs` with headless option support
- Frame transformation logic exists in `page.evaluate` calls calculating position and scale
- Both `setup.mjs` and `get-box.mjs` utility files exist in `scripts/e2e/utils/` directory
- All E2E tests use `page.goto(url)` for page navigation rather than alternative navigation methods

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations of R-PUPPETEER-001 MUST be caught during code review and CI pipeline checks. CI builds MUST fail if E2E tests do not use Puppeteer or bypass frame transformation logic.
</enforcement>