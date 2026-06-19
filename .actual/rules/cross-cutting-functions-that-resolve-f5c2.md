# Use console.warn for Component Resolution Failures in Data Resolution Functions: Functions That Resolve

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector, specifically resolveAndReplaceData, resolveDataById, and resolveDataBySelector, as well as any new data resolution utilities that query the component store.

### Rules

- **R-RESOLVE-001** SHOULD: Functions that resolve component data (resolveAndReplaceData, resolveDataById, resolveDataBySelector) SHOULD use console.warn with a standardized message format when component resolution fails.
- **R-RESOLVE-002** MUST: Use the exact message format: 'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.' for ID-based lookups.
- **R-RESOLVE-003** MUST: For selector-based lookups, serialize the selector with JSON.stringify to provide complete diagnostic information in the warning message.
- **R-RESOLVE-004** MUST: Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic.
- **R-RESOLVE-005** MUST: Ensure the warning message includes enough context (ID, selector, or function name) for developers to trace the issue back to the calling code.
- **R-RESOLVE-006** MUST NOT: Throw exceptions for component resolution failures in these functions; allow the application to continue operating when components are dynamically removed.

### Verify

```bash
# Verify all three data resolution functions contain console.warn calls
grep -r 'console.warn.*Could not find component' packages/core/lib/data/

# Count occurrences of the three main resolution functions
grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l

# Verify the three required files exist
test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format.
- Warning messages include the specific identifier (ID or selector) that failed to resolve.
- No exceptions are thrown for component resolution failures in these functions.
- The warning message format matches the specified template with proper context inclusion.

<enforcement>
Claude Code MUST NOT skip or defer verification. All three data resolution functions must be checked for compliance with the console.warn pattern before accepting changes to this scope.
</enforcement>