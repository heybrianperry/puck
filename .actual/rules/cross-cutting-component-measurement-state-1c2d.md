# Adopt Map-Based Caching for Component Measurement State in Public API Components: Component Measurement State

These rules are ALWAYS ACTIVE for all public API components in packages/core that manage component measurement state, virtualization handles, and configuration retrieval patterns using Map-based caches and URLSearchParams.

### Rules

- **R-CACHE-001** MUST: Component measurement state (heights, refs, handles) MUST be stored in Map instances using component identifiers as keys.
- **R-CACHE-002** MUST: Every .set() operation on component-scoped Map caches MUST have a corresponding .delete() in a useEffect cleanup function or component unmount path.
- **R-CACHE-003** MUST: Map instances MUST be initialized at component scope (not inside hooks) to ensure cache stability across re-renders.
- **R-CACHE-004** MUST: Cleanup operations MUST be implemented in useEffect return functions: `return () => { measuredItemHeights.delete(componentId); measureRefsRef.current.delete(componentId); }`
- **R-CACHE-005** MUST: URLSearchParams caching in client components MUST be guarded with typeof window checks: `const params = typeof window === 'undefined' ? new URLSearchParams() : new URL(window.location.href).searchParams`
- **R-CACHE-006** SHOULD: Cache key formats and lifecycle expectations SHOULD be documented in component API documentation to ensure consistent usage across public API consumers.
- **R-CACHE-007** SHOULD: Cache operations SHOULD be coordinated with React hooks: useEffect for cleanup (.delete()), useCallback for cache-dependent callbacks, and useMemo for derived cache values.

### Verify

```bash
# Verify Map-based cache .get() operations in public API components
grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'

# Verify Map-based cache .set() operations
grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'

# Verify Map-based cache .delete() cleanup operations
grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'

# Verify URLSearchParams usage in demo applications
grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'
```

**Accept when:**
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards
- Map instances are initialized at component scope, not inside hooks
- Cache key formats and lifecycle expectations are documented in component API documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules are mandatory for public API components managing measurement state. Violations detected by grep-based verification in CI pipeline MUST block merge. Code review MUST verify Map cleanup in useEffect return functions before approval. Integration tests MUST validate cache cleanup after component unmount.
</enforcement>