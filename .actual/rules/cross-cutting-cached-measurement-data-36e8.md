# Adopt Map-Based Caching for Component Measurement State in Public API Components: Cached Measurement Data

These rules are ALWAYS ACTIVE for all public API components in packages/core that manage component measurement state, virtualization handles, and configuration parameters using Map-based caches coordinated with React hooks.

### Rules

- **R-CACHE-001** MUST: Cached measurement data MUST be cleaned up via .delete() when components unmount or identifiers change.
- **R-CACHE-002** MUST: Initialize Map instances at component scope (not inside hooks) to ensure cache stability across re-renders.
- **R-CACHE-003** MUST: Implement cleanup in useEffect return functions that call .delete() for all Map-based caches when components unmount.
- **R-CACHE-004** MUST: Every .set() operation on component-scoped Map caches MUST have a corresponding .delete() in a useEffect cleanup function or component unmount path.
- **R-CACHE-005** SHOULD: Coordinate cache operations with React hooks using useEffect for cleanup, useCallback for cache-dependent callbacks, and useMemo for derived cache values.
- **R-CACHE-006** SHOULD: For URLSearchParams caching in client components, guard with typeof window checks to ensure SSR compatibility.
- **R-CACHE-007** SHOULD: Document cache key formats and lifecycle expectations in component API documentation to ensure consistent usage across public API consumers.

### Verify

```bash
# Verify Map-based cache .get() operations in public API components
grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'

# Verify Map-based cache .set() operations in public API components
grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'

# Verify Map-based cache .delete() operations in cleanup paths
grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'

# Verify URLSearchParams usage in demo applications with window guards
grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'
```

**Accept when:**
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path
- Map instances are initialized at component scope, not inside React hooks
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards
- Cache cleanup is verified in useEffect return functions for all component unmount paths

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All Map-based cache operations in public API components must be verified to have corresponding cleanup logic before code review approval.
</enforcement>