# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Each Virtualized Zone

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and components requiring dynamic item height measurement with interaction handlers accessing measurement data across multiple virtualized zones.

### Rules

- **R-VSCROLL-001** MUST: Each virtualized zone MUST maintain an independent measurement cache identified by a unique compound key (zoneCompound).
- **R-VSCROLL-002** MUST: Use `useRef` to create Map instances for measurement caches: `const measuredItemHeights = useRef(new Map())`.
- **R-VSCROLL-003** MUST: Access cache within callbacks using `.current`: `measuredItemHeights.current.get(componentId)` and `measuredItemHeights.current.set(componentId, height)`.
- **R-VSCROLL-004** MUST: Implement cleanup in useEffect return to delete cache entries: `return () => { measuredItemHeights.current.delete(componentId); }`.
- **R-VSCROLL-005** SHOULD: For multi-zone coordination, maintain a root-level Map of virtualizer handles keyed by zone compound identifiers.
- **R-VSCROLL-006** MAY: Consider using WeakMap if component IDs are object references to enable automatic garbage collection.

### Verify

```bash
# Detect Map-based measurement cache operations in virtualized components
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect useRef with Map pattern
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect root virtualizer cache operations
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal
- Each virtualized zone maintains independent measurement caches with unique compound keys
- Scroll performance meets or exceeds 60fps threshold in performance testing

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching patterns in virtualized components. All grep patterns MUST execute successfully and return results matching the expected cache operation signatures.
</enforcement>