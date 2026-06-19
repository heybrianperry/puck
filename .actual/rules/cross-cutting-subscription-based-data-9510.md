# Standardize React Component Export as Public API Contracts in Data Access Layer: Subscription Based Data

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, components using Map-based caching for component-scoped state, components coordinating data access through hooks, and components integrating with external libraries for data-driven UI interactions.

### Rules

- **R-SUB-001** SHOULD: Subscription-based data access patterns SHOULD use store abstractions with explicit cleanup in useEffect hooks.
- **R-SUB-002** MUST: All public API components MUST export TypeScript type definitions for their props interfaces.
- **R-SUB-003** MUST: Map-based caching operations (get/set/delete) MUST be wrapped in useCallback or useMemo hooks to maintain referential stability.
- **R-SUB-004** MUST: useEffect hooks that subscribe to stores or set up event listeners MUST return cleanup functions.
- **R-SUB-005** SHOULD: Array.find() operations in data access paths SHOULD be converted to memoized hooks or Map structures for O(1) lookup performance.
- **R-SUB-006** SHOULD: Cache eviction strategies SHOULD be documented in component documentation when using Map-based caching for unbounded datasets.

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
- useEffect cleanup functions are present for all subscriptions and side effects

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block PR merges for missing type exports. Code review feedback requires refactoring of direct array.find() usage in render paths to memoized hooks. Performance regression tests flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>