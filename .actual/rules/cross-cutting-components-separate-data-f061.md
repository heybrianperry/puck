# Standardize React Component Export as Public API Contracts in Data Access Layer: Components Separate Data

These rules are ALWAYS ACTIVE for all React components in packages/core/components that export public APIs, particularly those using Map-based caching for component-scoped state and coordinating data access through hooks.

### Rules

- **R-DAL-001** SHOULD: Components SHOULD separate data access logic from presentation through custom hooks or context providers (e.g., useFrame, autoFrameContext).
- **R-DAL-002** MUST: All public API components export TypeScript type definitions for their props interfaces using export type syntax.
- **R-DAL-003** MUST: Map-based caching operations (get/set/delete) for component-scoped state MUST be wrapped in useCallback or useMemo hooks to maintain referential stability.
- **R-DAL-004** MUST: All useEffect hooks that subscribe to stores or set up event listeners MUST return cleanup functions (e.g., return () => rootVirtualizers.delete(zoneCompound)).
- **R-DAL-005** SHOULD: Array.find() operations in data access paths SHOULD be converted to Map structures for O(1) lookup performance or wrapped in useMemo hooks.
- **R-DAL-006** MUST: ESLint rules MUST enforce useEffect cleanup functions for all subscriptions and side effects.

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
- Map-based caching operations (get/set/delete) are present in components coordinating data access and wrapped in hooks
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation
- All useEffect hooks with subscriptions include cleanup functions
- Array.find() operations in render paths are memoized or converted to Map-based lookups

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures for missing type exports block PR merges. Code review checklist MUST verify Map-based caching patterns and hook usage. ESLint rules MUST enforce useEffect cleanup functions. Performance regression tests MUST flag components with uncached data access patterns exceeding budget thresholds.
</enforcement>