# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: E2e Tests Public

These rules are ALWAYS ACTIVE for all E2E tests validating public/external API contracts, browser automation for UI component testing, element inspection across iframe boundaries, and coordinate/scale transformation calculations in the scripts/e2e/ directory.

### Rules

- **R-E2E-001** MUST: E2E tests for public API contracts MUST use Puppeteer as the browser automation library.

### Verify

```bash
# Verify Puppeteer launch configuration with headless support
grep -r 'puppeteer.launch' scripts/e2e/ | grep -q 'headless'

# Verify frame transformation logic in page.evaluate calls
grep -r 'page.evaluate' scripts/e2e/ | grep -q 'getFrameTransform'

# Verify utility files exist
test -f scripts/e2e/utils/setup.mjs && test -f scripts/e2e/utils/get-box.mjs
```

**Accept when:**
- Puppeteer launch configuration is present in scripts/e2e/utils/setup.mjs with headless option support
- Frame transformation logic exists in page.evaluate calls calculating position and scale
- Both setup.mjs and get-box.mjs utility files exist in scripts/e2e/utils/ directory
- E2E tests use Puppeteer for browser automation without bypassing frame transformation logic

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations trigger CI build failure and code review blocks until Puppeteer usage patterns and frame-aware element inspection are confirmed.
</enforcement>