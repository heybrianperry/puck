# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Include

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-WARN-001** MUST: Warning messages MUST include the specific identifier (component ID or selector) that failed to resolve.
- **R-WARN-002** MUST: Use console.warn (not console.error) for component resolution failures in data resolution functions.
- **R-WARN-003** MUST: Use the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- **R-WARN-004** MUST: For selector-based lookups, serialize the selector using JSON.stringify() to ensure complex objects are readable in the warning message.
- **R-WARN-005** MUST: Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic.

### Verify

```bash
# Count console.warn calls in data resolution modules with expected message pattern
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error calls are used for component resolution failures
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Check for component resolution warning tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Warning messages follow the exact format specified in R-WARN-003
- Selectors are serialized with JSON.stringify() in warning output

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All data resolution functions in scope MUST emit console.warn with the required message format and identifier inclusion.
</enforcement>