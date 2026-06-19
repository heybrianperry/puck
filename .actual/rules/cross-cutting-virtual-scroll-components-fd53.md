# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Virtual Scroll Components

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, including VirtualizedDropZone, DragDropContext, and related drag-and-drop contexts requiring measurement tracking across dynamic list operations.

### Rules

- **R-VSCROLL-001** MUST: Virtual scroll components MUST use Map data structures to cache item measurements keyed by component ID.

### Verify

```bash
# Count Map-based measurement caching operations
grep -r 'measuredItemHeights\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map-based measurement refs operations
grep -r 'measureRefsRef\.current\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map instantiations in virtualized/cached contexts
grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l
```

**Accept when:**
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual
- Measurement Maps are initialized using useRef to maintain reference stability across renders
- Map.has() is used to check for cache hits before calling get() to distinguish between cached undefined values and cache misses

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based measurement caching patterns in virtual scroll components. All new virtual scroll components integrating with @tanstack/react-virtual MUST implement Map-based caching or document an approved exception.
</enforcement>