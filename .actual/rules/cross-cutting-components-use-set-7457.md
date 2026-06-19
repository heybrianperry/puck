# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Components Use Set

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and components requiring dynamic item height measurement with interaction handlers accessing measurement data across multi-zone virtualized layouts.

### Rules

- **R-VSCROLL-001** MAY: Components MAY use Set data structures for tracking pinned indexes or other interaction state alongside measurement caches.

### Verify

```bash
# Detect Map-based measurement caching in virtualized components
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect useRef with Map initialization
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Detect root virtualizer cache operations
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal
- Set data structures are used for tracking pinned indexes or interaction state alongside measurement caches

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching patterns in virtual scroll components. All three grep patterns MUST be executed to confirm adoption across the codebase.
</enforcement>