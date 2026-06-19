# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Components Clean Measurement

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, requiring efficient measurement caching for dynamic item heights and drag-and-drop coordination.

### Rules

- **R-MEAS-001** SHOULD: Components SHOULD clean up measurement cache entries using delete operations when items are removed from the virtual list.

### Verify

```bash
# Count Map-based measurement cache operations
grep -r 'measuredItemHeights\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map-based measurement refs operations
grep -r 'measureRefsRef\.current\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map initializations in measurement/cache contexts
grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l
```

**Accept when:**
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual
- Measurement cache entries are explicitly deleted when items are removed from the virtual list

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All virtual scroll components must demonstrate explicit cache cleanup via Map.delete operations in unmount or removal handlers.
</enforcement>