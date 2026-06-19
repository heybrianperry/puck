# Adopt Map-Based Caching for Component Measurement State in Public API Components: Cache Operations Use

These rules are ALWAYS ACTIVE for all public API components in packages/core that manage component measurement state, virtualization handles, and configuration parameters using Map-based caches, as well as client-side configuration retrieval in demo applications using URLSearchParams.

### Rules

- **R-CACHE-001** MUST: Cache operations MUST use .get(), .set(), and .delete() methods for retrieval, storage, and cleanup of cached values.

### Verify

```bash
# Verify .get() operations on component-scoped caches
grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'

# Verify .set() operations on component-scoped caches
grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'

# Verify .delete() operations in cleanup paths
grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'

# Verify URLSearchParams.get() usage in demo applications
grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'
```

**Accept when:**
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards
- Map instances are initialized at component scope (not inside hooks) to ensure cache stability across re-renders
- Cache operations are coordinated with React hooks: useEffect for cleanup, useCallback for cache-dependent callbacks, and useMemo for derived cache values

<enforcement>
Claude Code MUST NOT skip or defer verification of R-CACHE-001. All Map-based cache operations in public API components must follow the .get()/.set()/.delete() pattern with corresponding cleanup logic in useEffect return functions. Violations block merge and trigger CI pipeline failures.
</enforcement>