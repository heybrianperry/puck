# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Virtualizer Handle Registries

These rules are ALWAYS ACTIVE for all virtual scroll and drag-drop components in `packages/core/components` that manage measurement caches and virtualizer handle registries using React refs.

### Rules

- **R-VIRT-001** MUST: Virtualizer handle registries (rootVirtualizers) MUST use Map data structures keyed by component identifiers (componentId, zoneCompound) to support O(1) lookup and deletion.
- **R-VIRT-002** MUST: Initialize measurement caches as `useRef(new Map())` at component top level to ensure single Map instance per component lifecycle.
- **R-VIRT-003** MUST: Wrap all cache access operations (get, set, delete) in `useCallback` hooks to maintain referential stability for child component props.
- **R-VIRT-004** MUST: Structure `useEffect` cleanup functions to iterate over all registered keys and call `.delete()` before component unmount.
- **R-VIRT-005** MUST: Use TypeScript generics to type Map keys and values (e.g., `Map<string, number>` for measuredItemHeights) to prevent runtime type errors during cache operations.
- **R-VIRT-006** MUST: Every `Map.set()` operation MUST have a corresponding `Map.delete()` in a `useEffect` cleanup function or unmount handler.
- **R-VIRT-007** SHOULD: Restrict cache mutations to `useEffect` and `useCallback` scopes; document synchronization requirements in component interfaces.
- **R-VIRT-008** SHOULD: Implement ResizeObserver integration to trigger cache invalidation when component dimensions change due to external factors.

### Verify

```bash
# Verify useRef(new Map()) pattern usage
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify cleanup functions call .delete()
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count Map operations
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l
```

**Accept when:**
- All virtual scroll components use `useRef(new Map())` for measurement caches and virtualizer registries
- Every `Map.set()` operation has a corresponding `Map.delete()` in a `useEffect` cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- TypeScript types are applied to all Map declarations (no untyped `Map` instances)
- No measurement cache operations occur outside `useEffect` or `useCallback` scopes

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if Map-based caches lack cleanup functions. CI pipeline MUST fail if grep verification commands do not match expected pattern counts. Runtime warnings MUST be logged in development mode when cache size exceeds threshold after component unmount.
</enforcement>