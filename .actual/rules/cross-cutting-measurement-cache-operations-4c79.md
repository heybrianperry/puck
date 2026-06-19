# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Measurement Cache Operations

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and components requiring dynamic item height measurement with interaction handlers accessing measurement data across multiple virtualized zones.

### Rules

- **R-MEAS-001** MUST: Measurement cache operations (get, set, delete) MUST be performed within React refs to avoid triggering component re-renders.

### Verify

```bash
# Detect Map-based measurement caching in virtualized components
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect useRef with Map initialization patterns
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect root virtualizer cache operations
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal
- Cache is accessed via `.current` property within callbacks: `measuredItemHeights.current.get(componentId)`
- useEffect cleanup functions implement cache deletion: `return () => { measuredItemHeights.current.delete(componentId); }`

<enforcement>
Claude Code MUST NOT skip or defer verification of measurement cache operations in virtualized components. All three grep patterns MUST be executed and results reviewed to confirm Map-based caching is properly implemented in refs.
</enforcement>