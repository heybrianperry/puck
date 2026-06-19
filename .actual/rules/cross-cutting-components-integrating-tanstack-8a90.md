# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Components Integrating Tanstack

These rules are ALWAYS ACTIVE for all virtual scroll and drag-drop components in `packages/core` that integrate `@tanstack/react-virtual` and `@dnd-kit` libraries.

### Rules

- **R-VIRT-001** MUST: Components integrating @tanstack/react-virtual MUST store measurement data (measuredItemHeights) and measurement refs (measureRefsRef) in mutable ref containers accessed via .current
- **R-VIRT-002** MUST: Initialize measurement caches as useRef(new Map()) at component top level to ensure single Map instance per component lifecycle
- **R-VIRT-003** MUST: Wrap all cache access operations (get, set, delete) in useCallback hooks to maintain referential stability for child component props
- **R-VIRT-004** MUST: Structure useEffect cleanup functions to iterate over all registered keys and call .delete() before component unmount
- **R-VIRT-005** MUST: Use TypeScript generics to type Map keys and values (e.g., Map<string, number> for measuredItemHeights) to prevent runtime type errors during cache operations
- **R-VIRT-006** SHOULD: Implement ResizeObserver integration to trigger cache invalidation when component dimensions change due to external factors
- **R-VIRT-007** SHOULD: Restrict cache mutations to useEffect and useCallback scopes; document synchronization requirements in component interfaces

### Verify

```bash
# Verify useRef(new Map()) pattern for measurement caches
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify cleanup functions call .delete() in useEffect
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count Map operations to verify pattern adoption
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l
```

**Accept when:**
- All virtual scroll components use useRef(new Map()) for measurement caches and virtualizer registries
- Every Map.set() operation has a corresponding Map.delete() in a useEffect cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- All measurement cache Map instances are typed with TypeScript generics
- Cache access operations are wrapped in useCallback hooks

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if Map-based caches lack cleanup functions. CI pipeline MUST fail if grep verification commands do not match expected pattern counts. Runtime warnings MUST be logged in development mode when cache size exceeds threshold after component unmount.
</enforcement>