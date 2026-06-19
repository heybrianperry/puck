# Standardize React Component Export as Public API Contracts in Data Access Layer: React Components Serving

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, components using Map-based caching for component-scoped state, components coordinating data access through hooks, and components integrating with external libraries for data-driven UI interactions.

### Rules

- **R-DAL-001** MUST: React components serving as public API contracts MUST be exported with explicit TypeScript types or interfaces (e.g., VirtualizedDropZone, DragDropContext, AutoFrameProps).
- **R-DAL-002** MUST: Map-based caching operations (get/set/delete) for component-scoped state MUST be wrapped in useCallback or useMemo hooks to maintain referential stability and prevent unnecessary re-renders.
- **R-DAL-003** MUST: All useEffect hooks that subscribe to stores or set up event listeners MUST return cleanup functions (e.g., return () => rootVirtualizers.delete(zoneCompound)).
- **R-DAL-004** SHOULD: Array.find() operations in data access paths SHOULD be converted to Map structures for O(1) lookup performance or wrapped in useMemo to prevent performance degradation.
- **R-DAL-005** SHOULD: Component documentation SHOULD include cache eviction strategies when using Map-based caching for unbounded datasets.
- **R-DAL-006** MAY: Legacy components with established external consumers MAY defer TypeScript type exports until the next major version (EXC-001).
- **R-DAL-007** MAY: Performance-critical paths MAY expose internal data structures if profiling demonstrates measurable benefit greater than 10% improvement (EXC-002).

### Verify

```bash
# Count exported Props types in public API components
grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l

# Count Map-based caching operations for component-scoped state
grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l

# Count React hook usage for data access encapsulation
grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l
```

**Accept when:**
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access and wrapped in hooks
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- All useEffect subscriptions include cleanup functions

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compilation failures for missing type exports block PR merges. Code review must verify Map-based caching patterns, hook usage, and useEffect cleanup functions. Performance regression tests flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>