# Use console.warn for Component Resolution Failures in Data Resolution Functions: Component Resolution Failures

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-CORE-001** MUST NOT: Component resolution failures MUST NOT throw exceptions or use console.error unless the failure represents an unrecoverable error.
- **R-CORE-002** MUST: Use console.warn with the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- **R-CORE-003** MUST: Include the failed identifier (ID or serialized selector) in the warning message to provide actionable diagnostic context.
- **R-CORE-004** MUST: For selector-based lookups, serialize the selector using JSON.stringify() to ensure complex objects are readable in the warning message.
- **R-CORE-005** MUST: Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic.
- **R-CORE-006** SHOULD: Apply this pattern consistently across all three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector).

### Verify

```bash
# Verify console.warn usage for component resolution failures
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error calls for component resolution in data resolution modules
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Verify warning messages in component resolution tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Warning messages follow the exact format specified in R-CORE-002
- Complex selectors are serialized using JSON.stringify() for readability

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All data resolution functions in scope MUST comply with R-CORE-001 through R-CORE-006 before code review approval.
</enforcement>