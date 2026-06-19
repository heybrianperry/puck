# Use console.warn for Component Resolution Failures in Data Resolution Functions: Component Resolution Failures

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-CORE-001** MUST: Component resolution failures in data resolution functions MUST log warnings using console.warn
- **R-CORE-002** MUST: Warning messages MUST include the failed identifier (ID or serialized selector) and explain potential causes
- **R-CORE-003** MUST: Use the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- **R-CORE-004** MUST: For selector-based lookups, serialize the selector using JSON.stringify() to ensure complex objects are readable in the warning message
- **R-CORE-005** MUST: Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic
- **R-CORE-006** MUST: No console.error calls are permitted for component resolution failures in data resolution modules

### Verify

```bash
# Verify console.warn usage for component resolution failures
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error calls in data resolution modules
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Verify warning messages in tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Warning messages follow the exact format specified in R-CORE-003
- Selectors are serialized using JSON.stringify() in warning messages
- console.warn calls appear immediately after component resolution failure detection

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-CORE rules are mandatory for data resolution functions in packages/core/lib/data/.
</enforcement>