# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: E2e Utility Functions

These rules are ALWAYS ACTIVE for all E2E tests validating public/external API contracts, browser automation for UI component testing, element inspection across iframe boundaries, and coordinate/scale transformation calculations in the scripts/e2e/ directory.

### Rules

- **R-E2E-001** SHOULD: E2E utility functions SHOULD be organized in scripts/e2e/utils/ directory
- **R-E2E-002** MUST: Initialize Puppeteer with puppeteer.launch({ headless, ...options }) allowing configuration override for debugging
- **R-E2E-003** MUST: Implement getFrameTransform recursively to handle arbitrary iframe nesting depth, accumulating position and scale transformations
- **R-E2E-004** MUST: Use page.evaluate() to execute frame inspection logic in browser context, returning serializable coordinate data
- **R-E2E-005** SHOULD: Structure E2E utilities as separate modules (setup.mjs, get-box.mjs) for reusability across test files
- **R-E2E-006** MUST: Handle null cases when elements are not found in either main document or frame contexts

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
- E2E tests use Puppeteer for browser automation and do not bypass frame transformation logic

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations trigger CI build failure and code review blocks until frame-aware element inspection patterns are properly implemented.
</enforcement>