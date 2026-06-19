# Adopt Map-Based Caching for Component Measurement State in Public API Components: Map Based Caches

These rules are ALWAYS ACTIVE for all public API components in packages/core that manage component measurement state, virtualization handles, and configuration parameters using Map-based caches coordinated with React hooks.

### Rules

- **R-MAP-001** SHOULD: Map-based caches SHOULD be coordinated with React hooks (useCallback, useMemo, useEffect) to ensure cache operations occur at appropriate lifecycle points.
- **R-MAP-002** MUST: Initialize Map instances at component scope (not inside hooks) to ensure cache stability across re-renders.
- **R-MAP-003** MUST: Implement cleanup in useEffect return functions to call .delete() for all Map-based caches when components unmount or identifiers change.
- **R-MAP-004** MUST: Every .set() operation on component-scoped Map caches MUST have a corresponding .delete() in a useEffect cleanup function or component unmount path.
- **R-MAP-005** SHOULD: Use consistent .get() retrieval patterns across both Map instances and URLSearchParams to establish uniform key-value access across the public API surface.
- **R-MAP-006** MUST: Guard URLSearchParams caching in client components with typeof window checks to ensure SSR compatibility.
- **R-MAP-007** SHOULD: Document cache key formats and lifecycle expectations in component API documentation to ensure consistent usage across public API consumers.

### Verify

```bash
# Verify Map-based cache .get() operations in public API components
grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'

# Verify Map-based cache .set() operations
grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'

# Verify Map-based cache .delete() cleanup operations
grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'

# Verify URLSearchParams usage in demo applications with typeof window guards
grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'
```

**Accept when:**
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path
- Map instances are initialized at component scope, not inside React hooks
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards
- Cache cleanup is verified in integration tests validating cache cleanup after component unmount in VirtualizedDropZone and DragDropContext

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if Map-based cache operations are detected without corresponding cleanup logic. Code review MUST block merge if new Map caches are introduced without documented lifecycle management. Runtime warnings MUST be emitted in development mode when cache size grows beyond expected thresholds.
</enforcement>