# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Virtualizer Handles Registered

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract, and components requiring dynamic item height measurement with interaction handlers accessing measurement data in multi-zone virtualized layouts.

### Rules

- **R-VIRT-001** SHOULD: Virtualizer handles SHOULD be registered in a root-level Map structure to coordinate multiple virtualized zones.

### Verify

```bash
# Check for Map-based measurement caching in virtualized components
grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for useRef with Map initialization
grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'

# Check for root-level virtualizer handle registration
grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal
- Root-level Map instances coordinate multiple virtualized zones by zone compound identifiers
- Cache access patterns use `.current` accessor for ref-stored Maps

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching patterns in virtualized components. All three grep verification commands MUST be executed and their results reviewed before accepting code changes to virtual scroll or drag-drop interaction components.
</enforcement>