# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Drag Drop Path

These rules are ALWAYS ACTIVE for virtual scroll and drag-drop components in the packages/core module that integrate @tanstack/react-virtual and @dnd-kit libraries, particularly VirtualizedDropZone and DragDropContext implementations.

### Rules

- **R-VSCROLL-001** SHOULD: Drag-drop path resolution SHOULD use Array.find with destructured path identifiers to locate source zones within virtualized hierarchies.
- **R-VSCROLL-002** MUST: Initialize measurement caches as useRef(new Map()) at component top level to ensure single Map instance per component lifecycle.
- **R-VSCROLL-003** MUST: Wrap all cache access operations (get, set, delete) in useCallback hooks to maintain referential stability for child component props.
- **R-VSCROLL-004** MUST: Structure useEffect cleanup functions to iterate over all registered keys and call .delete() before component unmount.
- **R-VSCROLL-005** MUST: Use TypeScript generics to type Map keys and values (e.g., Map<string, number> for measuredItemHeights) to prevent runtime type errors during cache operations.
- **R-VSCROLL-006** MUST: Every Map.set() operation MUST have a corresponding Map.delete() in a useEffect cleanup function or unmount handler.
- **R-VSCROLL-007** SHOULD: Restrict cache mutations to useEffect and useCallback scopes; document synchronization requirements in component interfaces.
- **R-VSCROLL-008** SHOULD: Implement ResizeObserver integration to trigger cache invalidation when component dimensions change due to external factors.

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
- All virtual scroll components use useRef(new Map()) for measurement caches and virtualizer registries
- Every Map.set() operation has a corresponding Map.delete() in a useEffect cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- TypeScript types are applied to all Map declarations with explicit key and value types
- No Map-based caches exist without corresponding cleanup logic in useEffect

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if Map-based caches lack cleanup functions. CI pipeline MUST fail if grep verification commands do not match expected pattern counts. Integration tests MUST assert measurement cache size returns to zero after component unmount.
</enforcement>