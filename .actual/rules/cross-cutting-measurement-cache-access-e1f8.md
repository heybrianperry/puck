# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Measurement Cache Access

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and components requiring dynamic item height measurement with interaction handlers accessing measurement data in multi-zone virtualized layouts.

### Rules

- **R-MEAS-001** SHOULD: Measurement cache access SHOULD occur within useCallback, useMemo, or useEffect hooks to optimize performance.

### Verify

```bash
# Detect Map-based measurement cache operations in virtualized components
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

<enforcement>
Clause Code MUST NOT skip or defer verification of measurement cache patterns in virtualized components. Violations require code review feedback requesting refactor to Map-based caching, performance regression testing, or architecture review for alternative approaches.
</enforcement>