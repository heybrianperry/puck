# Standardize React Component Export as Public API Contracts in Data Access Layer: Public Components Not

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, components using Map-based caching for component-scoped state, components coordinating data access through hooks, and components integrating with external libraries for data-driven UI interactions.

### Rules

- **R-DAL-001** MUST NOT: Public API components MUST NOT expose internal data structures (Map, Set, subscription handles) directly in their props or return types.
- **R-DAL-002** MUST: Use TypeScript's export type syntax for component props interfaces to ensure clear API contracts (e.g., export type AutoFrameProps = {...}).
- **R-DAL-003** MUST: Wrap Map.get/set operations in useCallback or useMemo hooks to maintain referential stability and prevent unnecessary re-renders.
- **R-DAL-004** MUST: Always return cleanup functions from useEffect hooks when subscribing to stores or setting up event listeners (e.g., return () => rootVirtualizers.delete(zoneCompound)).
- **R-DAL-005** SHOULD: For array.find() operations, consider converting frequently queried arrays to Map structures for O(1) lookup performance.
- **R-DAL-006** SHOULD: Document cache eviction strategies in component documentation when using Map-based caching for unbounded datasets.

### Verify

```bash
# Count exported Props type definitions
grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l

# Count Map-based caching operations (get/set/delete)
grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l

# Count React hook usage for data access encapsulation
grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l
```

**Accept when:**
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- No internal data structures (Map, Set, subscription handles) are exposed directly in component props or return types
- All useEffect hooks with subscriptions or side effects include proper cleanup functions

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block PR merges for missing type exports. Code review feedback requires refactoring of direct array.find() usage in render paths to memoized hooks. Performance regression tests flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>