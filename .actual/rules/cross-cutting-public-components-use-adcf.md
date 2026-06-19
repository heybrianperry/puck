# Adopt Map-Based Caching for Component Measurement State in Public API Components: Public Components Use

These rules are ALWAYS ACTIVE for all public API components in packages/core that manage component measurement state, virtualization handles, and configuration parameters, as well as demo applications that retrieve configuration from query strings.

### Rules

- **R-CACHE-001** SHOULD: Public API components SHOULD use URLSearchParams.get() for retrieving configuration parameters from query strings in client-side contexts.
- **R-CACHE-002** MUST: Component-scoped Map instances for measurement state (measuredItemHeights, measureRefsRef, rootVirtualizers) MUST be initialized at component scope, not inside hooks, to ensure cache stability across re-renders.
- **R-CACHE-003** MUST: Every .set() operation on component-scoped Map caches MUST have a corresponding .delete() in a useEffect cleanup function or component unmount path.
- **R-CACHE-004** MUST: URLSearchParams usage in client components MUST be guarded with typeof window checks to prevent SSR errors.
- **R-CACHE-005** SHOULD: Cache operations SHOULD be coordinated with React hooks: useEffect for cleanup (.delete()), useCallback for cache-dependent callbacks, and useMemo for derived cache values.
- **R-CACHE-006** SHOULD: Cache key formats and lifecycle expectations SHOULD be documented in component API documentation to ensure consistent usage across public API consumers.

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
- Map instances are initialized at component scope (not inside hooks) to ensure cache stability across re-renders
- Cache cleanup is implemented in useEffect return functions for all component-scoped Map caches

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST be executed before accepting changes to public API components or demo applications that involve Map-based caching or URLSearchParams usage. Code review MUST verify Map cleanup in useEffect return functions. CI pipeline MUST fail if Map-based cache operations are detected without corresponding cleanup logic.
</enforcement>