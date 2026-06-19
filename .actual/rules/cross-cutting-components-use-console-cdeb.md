# Standardize React Component Export as Public API Contracts in Data Access Layer: Components Use Console

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, components using Map-based caching for component-scoped state, components coordinating data access through hooks, and components integrating with external libraries for data-driven UI interactions.

### Rules

- **R-COMP-001** MAY: Components MAY use console.warn for non-critical data access failures (e.g., stylesheet loading, CSP violations) when graceful degradation is acceptable.

### Verify

```bash
# Verify TypeScript type exports for public API components
grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l

# Verify Map-based caching operations in data access coordination
grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l

# Verify React hooks usage for data access encapsulation
grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l
```

**Accept when:**
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- console.warn usage is limited to non-critical failures with graceful degradation paths
- useEffect cleanup functions are present for all subscriptions and side effects

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block PR merges for missing type exports. Code review feedback requires refactoring of direct array.find() usage in render paths to memoized hooks. Performance regression tests flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>