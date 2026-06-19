# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Item Height Measurements

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and any components requiring dynamic item height measurement with interaction handlers accessing measurement data across render cycles.

### Rules

- **R-VIRT-001** MUST: Item height measurements in virtualized components MUST be cached using Map data structures keyed by component identifier.

### Verify

```bash
# Check for Map-based measurement cache operations in virtualized components
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for useRef with Map initialization patterns
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for root-level virtualizer Map operations
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching (e.g., `const measuredItemHeights = useRef(new Map())`)
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal (e.g., in useEffect return functions)
- Multi-zone virtualized layouts maintain independent measurement caches with proper isolation

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching in virtualized components. All virtualized scroll components with dynamic item height measurement MUST implement this pattern before code review approval.
</enforcement>