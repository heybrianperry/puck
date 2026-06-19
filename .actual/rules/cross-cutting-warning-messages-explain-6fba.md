# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Explain

These rules are ALWAYS ACTIVE for all data resolution functions in packages/core/lib/data/ that look up components by ID or selector from a store.

### Rules

- **R-WARN-001** MUST: Warning messages MUST explain the potential causes of the failure (component removed or invalid identifier).

### Verify

```bash
# Verify console.warn usage in data resolution modules
grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"

# Verify no console.error is used for component resolution failures
grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l

# Verify warning messages in tests
npm test -- --grep "component resolution" 2>&1 | grep -i "warning"
```

**Accept when:**
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules
- Warning message format matches: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'

<enforcement>
Claude Code MUST NOT skip or defer verification of console.warn usage patterns in data resolution functions.
</enforcement>