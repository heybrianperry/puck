# Use console.warn for Component Resolution Failures in Data Resolution Functions: Component Resolution Failures

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector.

### Rules

- **R-CORE-001** MUST NOT: Component resolution failures MUST NOT throw exceptions or halt execution, allowing the application to continue with graceful degradation.
- **R-CORE-002** MUST: Use console.warn with the exact message format: `'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'`
- **R-CORE-003** MUST: For selector-based lookups, serialize the selector with JSON.stringify to provide complete diagnostic information in the warning message.
- **R-CORE-004** MUST: Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic.
- **R-CORE-005** MUST: Ensure the warning message includes enough context (ID, selector, or function name) for developers to trace the issue back to the calling code.
- **R-CORE-006** SHOULD: Apply this pattern consistently across all data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) and any new data resolution utilities that query the component store.

### Verify

```bash
# Verify console.warn pattern is used for component resolution failures
grep -r 'console.warn.*Could not find component' packages/core/lib/data/

# Count data resolution functions to ensure coverage
grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l

# Verify all three primary data resolution files exist
test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format.
- Warning messages include the specific identifier (ID or selector) that failed to resolve.
- No exceptions are thrown for component resolution failures in these functions.
- New data resolution utilities follow the same console.warn pattern with consistent message formatting.

<enforcement>
Claude Code MUST NOT skip or defer verification. All data resolution functions must be audited to confirm console.warn usage and absence of exception-throwing behavior for component lookup failures.
</enforcement>