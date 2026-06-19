# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: Public Contracts Under

These rules are ALWAYS ACTIVE for all E2E tests validating public/external API contracts using browser automation, element inspection across iframe boundaries, and coordinate/scale transformation calculations in the `scripts/e2e/` directory.

### Rules

- **R-PUPPETEER-001** SHOULD: Public API contracts under test SHOULD be named descriptively (e.g., 'setup', 'getBox').
- **R-PUPPETEER-002** MUST: E2E tests MUST use Puppeteer for browser automation and page navigation.
- **R-PUPPETEER-003** MUST: Frame-aware element inspection MUST account for iframe boundaries, coordinate transformations, and scaling factors.
- **R-PUPPETEER-004** MUST: Puppeteer launch configuration MUST include headless option support for CI and debugging scenarios.
- **R-PUPPETEER-005** MUST: Frame transformation logic MUST be implemented recursively to handle arbitrary iframe nesting depth.
- **R-PUPPETEER-006** MUST: E2E utilities MUST be structured as separate modules (setup.mjs, get-box.mjs) for reusability across test files.
- **R-PUPPETEER-007** MUST: Null cases MUST be handled when elements are not found in either main document or frame contexts.

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
- Puppeteer launch configuration is present in `scripts/e2e/utils/setup.mjs` with headless option support
- Frame transformation logic exists in `page.evaluate` calls calculating position and scale
- Both `setup.mjs` and `get-box.mjs` utility files exist in `scripts/e2e/utils/` directory
- E2E tests use Puppeteer for browser automation without bypassing frame transformation logic
- Public API contracts are named descriptively and follow established utility patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. CI build MUST fail if E2E tests do not use Puppeteer or bypass frame transformation logic. Code review MUST block merges introducing alternative browser automation libraries without architectural discussion.
</enforcement>