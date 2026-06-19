# Standardize React Component Export as Public API Contracts in Data Access Layer: Components Coordinating Data

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, particularly those using Map-based caching for component-scoped state and coordinating data access through hooks.

### Rules

- **R-DAL-001** MUST: Components coordinating data access MUST use Map-based caching for keyed lookups (e.g., measuredItemHeights.get/set, rootVirtualizers.set/delete) when managing component-scoped state.
- **R-DAL-002** MUST: All public API components MUST export TypeScript type definitions for their props interfaces using export type syntax.
- **R-DAL-003** MUST: Map.get/set operations MUST be wrapped in useCallback or useMemo hooks to maintain referential stability and prevent unnecessary re-renders.
- **R-DAL-004** MUST: All useEffect hooks that subscribe to stores or set up event listeners MUST return cleanup functions (e.g., return () => rootVirtualizers.delete(zoneCompound)).
- **R-DAL-005** SHOULD: Array.find() operations on frequently queried arrays SHOULD be converted to Map structures for O(1) lookup performance or wrapped in useMemo hooks.
- **R-DAL-006** SHOULD: Cache eviction strategies SHOULD be documented in component documentation when using Map-based caching for unbounded datasets.
- **R-DAL-007** MAY: Performance-critical paths MAY expose internal data structures if profiling demonstrates measurable benefit (>10% improvement).

### Verify

```bash
# Count exported Props type definitions
grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l

# Count Map-based caching operations in data access components
grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l

# Count React hook usage for encapsulation
grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l
```

**Accept when:**
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- useEffect cleanup functions are implemented for all subscriptions and side effects

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures for missing type exports block PR merges. Code review must verify Map-based caching patterns and hook usage. ESLint rules enforce useEffect cleanup functions. Performance regression tests flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>