# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Measurement Cache Operations

These rules are ALWAYS ACTIVE for virtual scroll and drag-drop components in packages/core that manage dynamic item height measurements and virtualizer handle coordination across React re-renders.

### Rules

- **R-MEAS-001** MUST: Measurement cache operations (get, set, delete) MUST be performed imperatively within useEffect cleanup functions or event handlers to avoid triggering React re-renders.

### Verify

```bash
# Verify useRef(new Map()) pattern for measurement caches
grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'

# Verify Map.delete() calls exist in useEffect cleanup
grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'

# Count total Map cache operations
grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l
```

**Accept when:**
- All virtual scroll components use `useRef(new Map())` for measurement caches and virtualizer registries
- Every `Map.set()` operation has a corresponding `Map.delete()` in a useEffect cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure
- Measurement cache operations are scoped exclusively to useCallback and useEffect contexts, never in render paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash commands must execute successfully and return expected pattern matches before accepting changes to virtual scroll or drag-drop components.
</enforcement>