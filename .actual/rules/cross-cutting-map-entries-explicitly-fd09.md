# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Map Entries Explicitly

These rules are ALWAYS ACTIVE for React components managing virtualized lists or grids with dynamic item counts, drag-and-drop contexts requiring per-zone or per-component state tracking, component measurement and layout systems using @tanstack/react-virtual or similar virtualization libraries, and store subscription patterns where components register/unregister handlers keyed by identifier.

### Rules

- **R-MAP-001** MUST: Map entries MUST be explicitly deleted using .delete(key) during component cleanup or unmount phases to prevent memory leaks.

### Verify

```bash
# Find Map instances used for component lifecycle tracking
grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'

# Verify .delete() calls exist in cleanup paths
grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2

# Verify cleanup logic in useEffect return functions
grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All Map instances used for component lifecycle tracking have corresponding .delete() calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map-based component lifecycle tracking MUST include explicit cleanup logic. CI build fails if memory profiling tests detect reference accumulation exceeding threshold. Code review blocks merge if Map cleanup logic is missing or incomplete.
</enforcement>