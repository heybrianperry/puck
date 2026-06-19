# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Components Encapsulate Measurement

These rules are ALWAYS ACTIVE for virtual scroll and drag-drop components in `packages/core/components` that manage dynamic item measurements and virtualizer handles using React refs and the @tanstack/react-virtual library.

### Rules

- **R-MEAS-001** SHOULD: Components SHOULD encapsulate measurement cache access patterns behind useCallback-wrapped functions to maintain referential stability across renders.
- **R-MEAS-002** MUST: Initialize measurement caches as `useRef(new Map())` at component top level to ensure a single Map instance per component lifecycle.
- **R-MEAS-003** MUST: Wrap all cache access operations (get, set, delete) in useCallback hooks to maintain referential stability for child component props.
- **R-MEAS-004** MUST: Structure useEffect cleanup functions to iterate over all registered keys and call `.delete()` before component unmount.
- **R-MEAS-005** SHOULD: Use TypeScript generics to type Map keys and values (e.g., `Map<string, number>` for measuredItemHeights) to prevent runtime type errors during cache operations.
- **R-MEAS-006** MUST: Every `Map.set()` operation MUST have a corresponding `Map.delete()` in a useEffect cleanup function or unmount handler.

### Verify

```bash
# Identify all useRef(new Map()) patterns for measurement caches
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify cleanup patterns exist for Map operations
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count total Map cache operations
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l
```

**Accept when:**
- All virtual scroll components use `useRef(new Map())` for measurement caches and virtualizer registries
- Every `Map.set()` operation has a corresponding `Map.delete()` in a useEffect cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- All measurement cache operations are wrapped in useCallback hooks
- TypeScript types are applied to Map declarations

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST verify useEffect cleanup functions call `.delete()` for all registered cache keys. CI pipeline MUST fail if grep verification commands do not match expected pattern counts. Merge MUST be blocked if Map-based caches lack cleanup functions. Runtime warnings MUST be logged in development mode when cache size exceeds threshold after component unmount.
</enforcement>