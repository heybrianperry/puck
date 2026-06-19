# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Components Use Set

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, requiring efficient measurement caching for dynamic item heights and drag-and-drop coordination.

### Rules

- **R-VSCROLL-001** MAY: Components MAY use Set data structures for tracking pinned indexes or other non-measurement state related to virtualization.

### Verify

```bash
# Count Map-based measurement caching operations
grep -r 'measuredItemHeights\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map-based measurement refs operations
grep -r 'measureRefsRef\.current\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map instantiations for measurement/cache/virtualizer patterns
grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l
```

**Accept when:**
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual
- Set data structures are used only for non-measurement state (pinned indexes, tracking flags) and not for primary measurement caching

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching patterns and Set usage for non-measurement state in virtual scroll components.
</enforcement>