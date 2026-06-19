# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Virtualizer Handles Registered

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, requiring efficient measurement caching and virtualizer handle coordination.

### Rules

- **R-VIRT-001** SHOULD: Virtualizer handles SHOULD be registered in a Map structure keyed by zone compound identifiers for multi-zone coordination.

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
- Virtualizer handles are registered using Map structures with compound zone identifiers for multi-zone coordination
- Measurement Maps are initialized using useRef to maintain reference stability across renders

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based measurement caching patterns in virtual scroll components. All new virtual scroll components integrating with @tanstack/react-virtual MUST use Map-based caching with explicit cleanup handlers.
</enforcement>