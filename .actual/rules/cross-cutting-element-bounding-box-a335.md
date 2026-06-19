# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: Element Bounding Box

These rules are ALWAYS ACTIVE for all E2E tests validating public/external API contracts, browser automation for UI component testing, element inspection across iframe boundaries, and coordinate and scale transformation calculations in the scripts/e2e/ directory.

### Rules

- **R-PUPPETEER-001** MUST: Element bounding box calculations MUST apply frame transformations to return accurate coordinates relative to the viewport.

### Verify

```bash
# Verify Puppeteer launch configuration with headless option support
grep -r 'puppeteer.launch' scripts/e2e/ | grep -q 'headless'

# Verify frame transformation logic exists in page.evaluate calls
grep -r 'page.evaluate' scripts/e2e/ | grep -q 'getFrameTransform'

# Verify both setup.mjs and get-box.mjs utility files exist
test -f scripts/e2e/utils/setup.mjs && test -f scripts/e2e/utils/get-box.mjs
```

**Accept when:**
- Puppeteer launch configuration is present in scripts/e2e/utils/setup.mjs with headless option support
- Frame transformation logic exists in page.evaluate calls calculating position and scale
- Both setup.mjs and get-box.mjs utility files exist in scripts/e2e/utils/ directory

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations trigger CI build failure and code review blocks until frame-aware element inspection is properly implemented.
</enforcement>