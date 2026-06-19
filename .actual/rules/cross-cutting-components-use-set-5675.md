# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Components Use Set

These rules are ALWAYS ACTIVE for virtual scroll and drag-drop components in `packages/core/components` that manage measurement caches and virtualizer handles using ref-based patterns with Map and Set data structures.

### Rules

- **R-VSCROLL-001** MAY: Components MAY use Set data structures (nextPinnedIndexes) for index tracking when order-independent membership testing is required.
- **R-VSCROLL-002** MUST: Initialize measurement caches as `useRef(new Map())` at component top level to ensure single Map instance per component lifecycle.
- **R-VSCROLL-003** MUST: Wrap all cache access operations (get, set, delete) in `useCallback` hooks to maintain referential stability for child component props.
- **R-VSCROLL-004** MUST: Structure `useEffect` cleanup functions to iterate over all registered keys and call `.delete()` before component unmount.
- **R-VSCROLL-005** MUST: Use TypeScript generics to type Map keys and values (e.g., `Map<string, number>` for measuredItemHeights) to prevent runtime type errors during cache operations.
- **R-VSCROLL-006** MUST: Every `Map.set()` operation must have a corresponding `Map.delete()` in a `useEffect` cleanup function or unmount handler.
- **R-VSCROLL-007** SHOULD: Restrict cache mutations to `useEffect` and `useCallback` scopes; document synchronization requirements in component interfaces.

### Verify

```bash
# Verify useRef(new Map()) patterns for measurement caches
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify Map.delete() operations are paired with useEffect cleanup
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count total Map operations to establish baseline
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l

# Verify Set usage for index tracking
grep -r 'new Set' packages/core/components --include='*.tsx' | grep -i 'index\|pinned'
```

**Accept when:**
- All virtual scroll components use `useRef(new Map())` for measurement caches and virtualizer registries
- Every `Map.set()` operation has a corresponding `Map.delete()` in a `useEffect` cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- Set data structures are used for order-independent membership testing (e.g., nextPinnedIndexes)
- TypeScript generics properly type all Map keys and values
- All cache access operations are wrapped in `useCallback` hooks

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-VSCROLL rules MUST be verified before accepting changes to virtual scroll or drag-drop components. Code review MUST block merge if Map-based caches lack cleanup functions or if Set usage violates order-independence requirements. CI pipeline MUST fail if grep verification commands do not match expected pattern counts.
</enforcement>