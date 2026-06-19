# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Use

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-WARN-001** SHOULD: Warning messages SHOULD use the prefix 'Warning:' to clearly distinguish them from errors
- **R-WARN-002** MUST: Use console.warn (not console.error) for component resolution failures in data resolution functions
- **R-WARN-003** MUST: Include the failed identifier (ID or serialized selector) in the warning message
- **R-WARN-004** MUST: Explain potential causes in the warning message (e.g., "Component may have been removed or the id/selector is invalid")
- **R-WARN-005** MUST: Use the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- **R-WARN-006** MUST: For selector-based lookups, serialize the selector using JSON.stringify() to ensure complex objects are readable
- **R-WARN-007** MUST: Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic

### Verify

```bash
# Count console.warn calls for component resolution in data resolution modules
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error calls are used for component resolution failures
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Check for warning messages in component resolution tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Warning messages follow the exact format specified in R-WARN-005
- Selectors are serialized with JSON.stringify() in warning messages
- console.warn calls appear immediately after component resolution failure detection

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All seven rules must be checked during code review and testing.
</enforcement>