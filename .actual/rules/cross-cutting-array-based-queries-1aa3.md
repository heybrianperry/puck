# Standardize React Component Export as Public API Contracts in Data Access Layer: Array Based Queries

These rules are ALWAYS ACTIVE for all React components in `packages/core/components` that export public APIs, particularly those using Map-based caching for component-scoped state and coordinating data access through React hooks.

### Rules

- **R-DAL-001** MUST: Array-based queries using `.find()` MUST be encapsulated within React hooks (`useCallback`, `useMemo`) to prevent unnecessary recomputation.
- **R-DAL-002** MUST: All public API components MUST export TypeScript type definitions for their props interfaces using `export type ComponentProps = {...}` syntax.
- **R-DAL-003** MUST: Map-based caching operations (`.get()`, `.set()`, `.delete()`) MUST be wrapped in `useCallback` or `useMemo` hooks to maintain referential stability across renders.
- **R-DAL-004** MUST: All `useEffect` hooks that subscribe to stores or set up event listeners MUST return cleanup functions (e.g., `return () => rootVirtualizers.delete(zoneCompound)`).
- **R-DAL-005** SHOULD: For frequently queried arrays, consider converting to Map structures for O(1) lookup performance instead of relying on `.find()` operations.
- **R-DAL-006** SHOULD: Document cache eviction strategies in component documentation when using Map-based caching for unbounded datasets.

### Verify

```bash
# Count exported Props type definitions
grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l

# Count Map-based caching operations on component-scoped state
grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l

# Count React hook usage for data access encapsulation
grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l
```

**Accept when:**
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- All useEffect hooks with subscriptions include proper cleanup functions
- Array.find() operations in render paths are wrapped in memoized hooks

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compilation failures for missing type exports block PR merges. Code review must verify Map-based caching patterns and hook usage. ESLint rules enforce useEffect cleanup functions. Performance regression tests flag uncached data access patterns exceeding budget thresholds.
</enforcement>