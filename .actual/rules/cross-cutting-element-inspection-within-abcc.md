# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: Element Inspection Within

These rules are ALWAYS ACTIVE for all E2E tests validating public/external API contracts using browser automation, specifically for element inspection across iframe boundaries in the scripts/e2e/ directory.

### Rules

- **R-PUPPETEER-001** MUST: Element inspection within iframes MUST calculate frame transformations including position (x, y) and scale (scaleX, scaleY).

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
- Puppeteer launch configuration is present in scripts/e2e/utils/setup.mjs with headless option support
- Frame transformation logic exists in page.evaluate calls calculating position and scale
- Both setup.mjs and get-box.mjs utility files exist in scripts/e2e/utils/ directory
- E2E tests use Puppeteer for browser automation without bypassing frame transformation logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All E2E tests involving iframe element inspection MUST implement frame-aware transformations using Puppeteer's page.evaluate() API. Violations detected in CI pipeline or code review MUST block merges unless approved by architecture review board with documented exception in ADR.
</enforcement>