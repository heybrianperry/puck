# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Use

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector, including resolveAndReplaceData, resolveDataById, resolveDataBySelector, and any new data resolution utilities that query the component store.

### Rules

- **R-WARN-001** MUST: Warning messages MUST use the prefix 'Warning: Could not find component' to maintain consistent log filtering and searchability.
- **R-WARN-002** MUST: Use console.warn (not console.error or exceptions) for component resolution failures to allow graceful degradation while providing developer visibility.
- **R-WARN-003** MUST: Include the specific identifier (ID or serialized selector) in each warning message to provide sufficient context for debugging.
- **R-WARN-004** MUST: Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic.

### Verify

```bash
# Verify all three data resolution functions contain console.warn with standardized format
grep -r 'console.warn.*Could not find component' packages/core/lib/data/

# Count data resolution function implementations
grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l

# Verify all three core files exist
test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format
- Warning messages include the specific identifier (ID or selector) that failed to resolve
- No exceptions are thrown for component resolution failures in these functions
- The exact message format follows: 'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'
- For selector-based lookups, the selector is serialized with JSON.stringify in the warning message

<enforcement>
Claude Code MUST NOT skip or defer verification. All three data resolution functions must be inspected to confirm console.warn usage with the standardized prefix and identifier context before accepting changes to this scope.
</enforcement>