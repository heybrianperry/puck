# Use console.warn for Component Resolution Failures in Data Resolution Functions: Selector Based Failures

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-SELECTOR-001** SHOULD: Selector-based failures SHOULD serialize the selector using JSON.stringify for readability.
- **R-SELECTOR-002** MUST: Use console.warn (not console.error) for component resolution failures to signal recoverable runtime conditions.
- **R-SELECTOR-003** MUST: Include the failed identifier (ID or serialized selector) and potential causes in the warning message.
- **R-SELECTOR-004** MUST: Use the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- **R-SELECTOR-005** MUST: Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic.

### Verify

```bash
# Count console.warn calls for component resolution in data resolution modules
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error calls are used for component resolution failures
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Check for component resolution warning messages in tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"

# Verify JSON.stringify usage for selector serialization
grep -r "JSON.stringify" packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Selector-based lookups serialize the selector using JSON.stringify() for readability
- Warning messages follow the exact format specified in R-SELECTOR-004
- console.warn calls are placed immediately after detecting the failure, before fallback logic

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-SELECTOR rules MUST be checked during code review and CI pipeline validation.
</enforcement>