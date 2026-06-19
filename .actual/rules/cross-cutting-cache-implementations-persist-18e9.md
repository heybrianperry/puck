# Use Map-Based Measurement Caching for Virtual Scroll Component State: Cache Implementations Persist

These rules are ALWAYS ACTIVE for virtual scrolling and drag-drop components in packages/core/components requiring dynamic measurement caching, including VirtualizedDropZone and DragDropContext implementations that coordinate with @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-CACHE-001** MUST: Cache implementations MUST persist across React re-renders using refs (useRef) to avoid triggering reactive updates
- **R-CACHE-002** MUST: Initialize Map caches using useRef to ensure single instance per component: `const measuredItemHeights = useRef(new Map())`
- **R-CACHE-003** MUST: Access cache within useCallback hooks for measurement operations to maintain referential stability
- **R-CACHE-004** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount or are removed from virtual lists
- **R-CACHE-005** SHOULD: Add cache size monitoring in development builds to detect potential memory leaks early
- **R-CACHE-006** SHOULD: Ensure componentId generation produces unique, stable identifiers and document componentId requirements in component interfaces

### Verify

```bash
# Detect Map-based cache patterns in virtual scroll components
grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/

# Verify cache operations (get/set/delete) are present for componentId-keyed measurements
grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/

# Run tests for cache and measurement behavior
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'
```

**Accept when:**
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded
- useRef initialization pattern is used for all measurement cache instances
- useCallback hooks wrap all cache access operations
- useEffect cleanup functions call delete on cache entries for unmounted components

<enforcement>
Claude Code MUST NOT skip or defer verification. All virtual scroll components must be audited for Map-based cache patterns before approval. Performance regression tests must pass at 60fps threshold for scroll and drag interactions.
</enforcement>