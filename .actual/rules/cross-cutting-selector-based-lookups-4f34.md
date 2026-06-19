# Use console.warn for Component Resolution Failures in Data Resolution Functions: Selector Based Lookups

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector, including resolveAndReplaceData, resolveDataById, and resolveDataBySelector.

### Rules

- **R-CORE-RESOLUTION-001** SHOULD: Selector-based lookups SHOULD serialize the selector using JSON.stringify in warning messages to provide complete diagnostic information.
- **R-CORE-RESOLUTION-002** MUST: Use console.warn (not console.error or exceptions) when component resolution fails to allow graceful degradation while providing developer visibility.
- **R-CORE-RESOLUTION-003** MUST: Use the exact message format: 'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'
- **R-CORE-RESOLUTION-004** MUST: Include the specific identifier (ID or serialized selector) in the warning message for debugging context.
- **R-CORE-RESOLUTION-005** MUST: Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic.

### Verify

```bash
# Verify all three data resolution functions contain console.warn calls
grep -r 'console.warn.*Could not find component' packages/core/lib/data/

# Count data resolution function implementations
grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l

# Verify all three required files exist
test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format
- Warning messages include the specific identifier (ID or selector) that failed to resolve
- No exceptions are thrown for component resolution failures in these functions
- Selector-based lookups serialize the selector using JSON.stringify in the warning message

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for data resolution functions in packages/core/lib/data/.
</enforcement>