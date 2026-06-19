# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Map Based State

These rules are ALWAYS ACTIVE for React components managing virtualized lists or grids with dynamic item counts, drag-and-drop contexts requiring per-zone or per-component state tracking, component measurement and layout systems using @tanstack/react-virtual or similar virtualization libraries, and store subscription patterns where components register/unregister handlers keyed by identifier.

### Rules

- **R-MAP-001** SHOULD: Map-based state management SHOULD be encapsulated within useEffect cleanup functions or explicit cleanup handlers to ensure deterministic lifecycle coupling.

### Verify

```bash
# Verify Map instances are used for component lifecycle tracking
grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'

# Verify .delete() calls exist in cleanup paths
grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2

# Verify useEffect cleanup patterns with Map.delete()
grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All Map instances used for component lifecycle tracking have corresponding .delete() calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times
- Map instances are wrapped in useRef() to persist across React render cycles
- Compound keys are used when managing state across multiple namespaces to prevent key collisions

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map-based state management patterns MUST include explicit cleanup logic in useEffect return functions or cleanup handlers. Code review MUST verify that every Map.set() operation has a corresponding Map.delete() in a cleanup path. Memory profiling tests MUST pass before merge.
</enforcement>