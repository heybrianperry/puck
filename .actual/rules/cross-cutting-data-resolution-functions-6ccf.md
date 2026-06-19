# Use console.warn for Component Resolution Failures in Data Resolution Functions: Data Resolution Functions

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector.

### Rules

- **R-DRF-001** MUST: Data resolution functions MUST log component lookup failures using console.warn when a component cannot be found by ID or selector.

### Verify

```bash
# Verify console.warn usage in data resolution functions
grep -r 'console.warn.*Could not find component' packages/core/lib/data/

# Count data resolution function implementations
grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l

# Verify all three core data resolution files exist
test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format
- Warning messages include the specific identifier (ID or selector) that failed to resolve
- No exceptions are thrown for component resolution failures in these functions
- Message format matches: 'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'
- For selector-based lookups, the selector is serialized with JSON.stringify in the warning message

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass and all accept criteria must be satisfied before approving changes to data resolution functions.
</enforcement>