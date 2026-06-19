# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Component Lifecycle State

These rules are ALWAYS ACTIVE for React components managing virtualized lists or grids with dynamic item counts, drag-and-drop contexts requiring per-zone or per-component state tracking, component measurement and layout systems using @tanstack/react-virtual or similar virtualization libraries, and store subscription patterns where components register/unregister handlers keyed by identifier.

### Rules

- **R-LIFECYCLE-001** MUST: Component lifecycle state MUST be stored in Map data structures keyed by component identifier (componentId or zoneCompound) to enable O(1) lookup and deletion operations.

### Verify

```bash
# Verify Map usage for component lifecycle tracking
grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'

# Verify cleanup logic exists for Map operations
grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2

# Verify useEffect cleanup patterns
grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All Map instances used for component lifecycle tracking have corresponding .delete() calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times
- Map instances are wrapped in useRef() to persist across React render cycles
- Cleanup logic is implemented in useEffect return functions or explicit cleanup handlers
- Compound keys are used when managing state across multiple namespaces to prevent key collisions

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map-based component lifecycle state MUST include corresponding cleanup logic. Memory profiling tests MUST pass before merge. Code review MUST verify cleanup logic for all Map usage.
</enforcement>