# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Component Measurement Caches

These rules are ALWAYS ACTIVE for React components managing virtualized lists or grids with dynamic item counts, drag-and-drop contexts requiring per-zone or per-component state tracking, component measurement and layout systems using @tanstack/react-virtual or similar virtualization libraries, and store subscription patterns where components register/unregister handlers keyed by identifier.

### Rules

- **R-MAP-001** SHOULD: Component measurement caches (measuredItemHeights) SHOULD use Map structures to support dynamic resizing and re-measurement without array reindexing overhead.
- **R-MAP-002** MUST: All Map instances used for component lifecycle tracking MUST have corresponding .delete() calls in cleanup paths (useEffect return functions or explicit cleanup handlers).
- **R-MAP-003** SHOULD: Wrap Map instances in useRef() to persist across React render cycles without triggering re-renders on mutation.
- **R-MAP-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) when managing state across multiple namespaces to prevent key collisions.
- **R-MAP-005** MAY: Expose Map.size in development builds for debugging and memory leak detection during testing.

### Verify

```bash
# Verify Map usage for component measurement caching
grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'

# Verify cleanup logic exists for Map operations
grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2

# Verify useEffect cleanup patterns with Map.delete()
grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All Map instances used for component lifecycle tracking have corresponding .delete() calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times
- Map instances are wrapped in useRef() to prevent re-render triggers on mutation
- Compound keys are used when managing state across multiple namespaces

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if Map cleanup logic is missing or incomplete. CI build MUST fail if memory profiling tests detect reference accumulation exceeding threshold. Runtime warnings MUST be present in development builds when Map.size() exceeds expected bounds.
</enforcement>