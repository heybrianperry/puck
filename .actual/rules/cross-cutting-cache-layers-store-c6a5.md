# Adopt Map-Based Caching for Component Measurement State in Public API Components: Cache Layers Store

These rules are ALWAYS ACTIVE for all public API components in `packages/core` that manage component measurement state, virtualization handles, and configuration retrieval patterns using Map-based caches and URLSearchParams.

### Rules

- **R-CACHE-001** MAY: Cache layers MAY store virtualizer handles (rootVirtualizers.set/delete) or measurement refs (measureRefsRef.current.set/get) depending on component requirements.
- **R-CACHE-002** MUST: Initialize Map instances at component scope (not inside hooks) to ensure cache stability across re-renders: `const measuredItemHeights = new Map()`.
- **R-CACHE-003** MUST: Implement cleanup in useEffect return functions: `return () => { measuredItemHeights.delete(componentId); measureRefsRef.current.delete(componentId); }`.
- **R-CACHE-004** MUST: Every .set() operation on component-scoped Map caches MUST have a corresponding .delete() in a useEffect cleanup function or component unmount path.
- **R-CACHE-005** SHOULD: Coordinate cache operations with React hooks: use useEffect for cleanup (.delete()), useCallback for cache-dependent callbacks, and useMemo for derived cache values.
- **R-CACHE-006** SHOULD: For URLSearchParams caching in client components, guard with typeof window checks: `const params = typeof window === 'undefined' ? new URLSearchParams() : new URL(window.location.href).searchParams`.
- **R-CACHE-007** SHOULD: Document cache key formats and lifecycle expectations in component API documentation to ensure consistent usage across public API consumers.

### Verify

```bash
# Verify Map-based cache .get() operations
grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'

# Verify Map-based cache .set() operations
grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'

# Verify Map-based cache .delete() operations
grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'

# Verify URLSearchParams usage in demo applications
grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'
```

**Accept when:**
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations.
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path.
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards.
- Map instances are initialized at component scope, not inside hooks.
- Cache cleanup is implemented in useEffect return functions for all component-scoped Map caches.

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST be executed before accepting changes to public API components or cache layer implementations. Code review MUST verify Map cleanup in useEffect return functions. CI pipeline MUST fail if Map-based cache operations are detected without corresponding cleanup logic.
</enforcement>