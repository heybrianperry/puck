# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Measurement Caches Support

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual and @dnd-kit/react, requiring persistent measurement state across re-renders.

### Rules

- **R-CACHE-001** MUST: Measurement caches MUST support get, set, and delete operations using component IDs as keys.
- **R-CACHE-002** MUST: Initialize measurement Maps using useRef to maintain reference stability across renders.
- **R-CACHE-003** MUST: Pair every set operation with a corresponding delete in useEffect cleanup or component unmount handler.
- **R-CACHE-004** MUST: Use Map.has() to check for cache hits before calling get() to distinguish between cached undefined values and cache misses.
- **R-CACHE-005** SHOULD: Use compound keys (e.g., zoneCompound) for multi-zone coordination that uniquely identify virtualizer instances across the component tree.
- **R-CACHE-006** SHOULD: Expose cache size metrics (map.size) in development mode to detect memory leaks during testing.
- **R-CACHE-007** MUST: Prohibit Map.forEach or Array.from(map) in render methods; use targeted get/set operations only.

### Verify

```bash
# Count measurement cache get/set/delete operations
grep -r 'measuredItemHeights\.(get|set|delete)' packages/core/components/ | wc -l

# Count measurement refs cache operations
grep -r 'measureRefsRef\.current\.(get|set|delete)' packages/core/components/ | wc -l

# Count Map initializations in measurement/cache contexts
grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l

# Verify useEffect cleanup patterns for Map.delete
grep -r 'useEffect.*=>.*{' packages/core/components/ | xargs grep -l 'map\.delete' | wc -l

# Flag non-Map measurement caching patterns
grep -r 'measuredItemHeights\s*=' packages/core/components/ | grep -v 'useRef(new Map())' | wc -l
```

**Accept when:**
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual
- Measurement Maps are initialized with useRef(new Map()) to maintain reference stability
- Map.forEach and Array.from(map) are not used in render methods or hot paths
- Cache size metrics are exposed in development mode for memory leak detection

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for virtual scroll components with drag-and-drop capabilities. Violations must be flagged during code review and blocked until refactored to comply with Map-based caching requirements.
</enforcement>