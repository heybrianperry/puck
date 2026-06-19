# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Implementations Use Set

These rules are ALWAYS ACTIVE for React components managing virtualized lists, drag-drop contexts, and component measurement systems that require efficient lifecycle-scoped reference tracking across render cycles.

### Rules

- **R-MAP-001** MAY: Implementations MAY use Set data structures (e.g., `nextPinnedIndexes.add()`) for tracking component collections where uniqueness and membership testing are primary operations.
- **R-MAP-002** MUST: All Map instances used for component lifecycle tracking MUST have corresponding `.delete()` calls in cleanup paths (useEffect return functions or explicit cleanup handlers).
- **R-MAP-003** SHOULD: Wrap Map instances in `useRef()` to persist across React render cycles without triggering re-renders on mutation.
- **R-MAP-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) when managing state across multiple namespaces to prevent key collisions.
- **R-MAP-005** MUST: Component identifiers MUST be globally unique across component types or instances; document componentId generation contracts and implement runtime assertions for duplicate key detection in development builds.

### Verify

```bash
# Verify Map usage in virtualized components
grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'

# Verify cleanup logic exists for Map operations
grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2

# Verify useEffect cleanup patterns
grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All Map instances used for component lifecycle tracking have corresponding `.delete()` calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times
- Map instances are wrapped in useRef() to prevent re-render triggers
- Compound keys are used consistently when managing state across multiple namespaces

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if Map cleanup logic is missing or incomplete. CI build MUST fail if memory profiling tests detect reference accumulation exceeding threshold. ESLint custom rules MUST detect Map.set() without corresponding .delete() in component scope.
</enforcement>