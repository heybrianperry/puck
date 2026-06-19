# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Include

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector, including resolveAndReplaceData, resolveDataById, resolveDataBySelector, and any new data resolution utilities that query the component store.

### Rules

- **R-WARN-001** MUST: Warning messages MUST include the specific identifier (ID or selector) that failed to resolve and indicate the component may have been removed or the identifier is invalid.
- **R-WARN-002** MUST: Use console.warn (not console.error or exceptions) for component resolution failures to allow graceful degradation while providing developer visibility.
- **R-WARN-003** MUST: Use the exact message format: `'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'`
- **R-WARN-004** MUST: For selector-based lookups, serialize the selector with JSON.stringify to provide complete diagnostic information in the warning message.
- **R-WARN-005** MUST: Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic.

### Verify

```bash
# Verify all three data resolution functions contain console.warn calls
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
- Selector-based warnings use JSON.stringify for serialization
- Warning calls appear immediately after lookup failure detection

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All data resolution functions in scope MUST comply with R-WARN-001 through R-WARN-005 before code review approval.
</enforcement>