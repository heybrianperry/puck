# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Cleanup Operations Call

These rules are ALWAYS ACTIVE for virtual scroll and drag-drop components in `packages/core/components` that use ref-based measurement caches and virtualizer registries with @tanstack/react-virtual and @dnd-kit integration.

### Rules

- **R-VSCROLL-001** MUST: Cleanup operations MUST call .delete() on measurement caches and virtualizer registries when components unmount or identifiers change.

### Verify

```bash
# Verify useRef(new Map()) patterns for measurement caches
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify .delete() calls are present in useEffect cleanup functions
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count total Map operations to establish baseline
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l
```

**Accept when:**
- All virtual scroll components use `useRef(new Map())` for measurement caches and virtualizer registries
- Every `Map.set()` operation has a corresponding `Map.delete()` in a `useEffect` cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- No measurement cache entries persist after component unmount

<enforcement>
Clause R-VSCROLL-001 verification is mandatory. Code review MUST block merge if Map-based caches lack cleanup functions. CI pipeline MUST fail if grep verification commands do not match expected pattern counts. Integration tests MUST assert measurement cache size returns to zero after component unmount.
</enforcement>