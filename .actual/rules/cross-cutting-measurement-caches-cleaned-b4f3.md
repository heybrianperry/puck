# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Measurement Caches Cleaned

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and any components requiring dynamic item height measurement with interaction handlers accessing measurement data.

### Rules

- **R-MEAS-001** MUST: Measurement caches MUST be cleaned up (delete operations) when components unmount or items are removed.

### Verify

```bash
# Check for Map-based measurement cache operations
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for useRef with Map pattern
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for root virtualizer cache operations
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal
- useEffect return functions contain cache deletion logic for unmount scenarios
- Multi-zone virtualized layouts maintain independent caches with proper isolation

<enforcement>
Claude Code MUST NOT skip or defer verification of measurement cache cleanup patterns in virtualized components. All three grep patterns MUST be executed and results reviewed to confirm Map-based caching with proper cleanup is implemented.
</enforcement>