# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Measurement Refs Stored

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, requiring efficient measurement caching for dynamic item heights and drag-and-drop coordination.

### Rules

- **R-MEAS-001** MUST: Measurement refs MUST be stored in a separate Map structure (measureRefsRef.current) to maintain reference stability across renders.
- **R-MEAS-002** MUST: Initialize measurement Maps using useRef to maintain reference stability across renders: `const measuredItemHeights = useRef(new Map())`.
- **R-MEAS-003** MUST: Pair every Map.set operation with a corresponding Map.delete in useEffect cleanup or component unmount handler.
- **R-MEAS-004** MUST: Use Map.get/set/delete operations for measurement caching with component IDs as keys in all virtual scroll components.
- **R-MEAS-005** SHOULD: Use Map.has() to check for cache hits before calling get() to distinguish between cached undefined values and cache misses.
- **R-MEAS-006** SHOULD: Use compound keys (e.g., zoneCompound) for multi-zone coordination that uniquely identify virtualizer instances across the component tree.
- **R-MEAS-007** SHOULD: Expose cache size metrics (map.size) in development mode to detect memory leaks during testing.
- **R-MEAS-008** MUST NOT: Use plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual.
- **R-MEAS-009** MUST NOT: Use Map.forEach or Array.from(map) in hot render paths; use targeted get/set operations only.

### Verify

```bash
# Count Map-based measurement caching operations
grep -r 'measuredItemHeights\.(get\|set\|delete)' packages/core/components/ | wc -l

# Count measureRefsRef.current Map operations
grep -r 'measureRefsRef\.current\.(get\|set\|delete)' packages/core/components/ | wc -l

# Count new Map() initializations in measurement/cache contexts
grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l

# Verify useRef wrapping of Map initialization
grep -r 'useRef(new Map())' packages/core/components/ | wc -l

# Check for useEffect cleanup patterns with map.delete
grep -r 'useEffect.*map\.delete' packages/core/components/ | wc -l
```

**Accept when:**
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual
- Measurement Maps are initialized with useRef to maintain reference stability across renders
- Map.has() is used to check for cache hits before calling get() in critical paths
- No Map iteration (forEach, Array.from) appears in hot render paths

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All virtual scroll components must be audited for Map-based measurement caching compliance before merge.
</enforcement>