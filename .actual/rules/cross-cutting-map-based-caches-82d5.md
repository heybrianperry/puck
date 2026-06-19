# Use Map-Based Measurement Caching for Virtual Scroll Component State: Map Based Caches

These rules are ALWAYS ACTIVE for virtual scrolling and drag-drop components in packages/core/components requiring dynamic measurement caching, including VirtualizedDropZone and DragDropContext components that coordinate with @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-CACHE-001** SHOULD: Map-based caches SHOULD be accessed within useCallback or useEffect hooks to coordinate with React lifecycle events.
- **R-CACHE-002** MUST: Initialize Map caches using useRef to ensure a single instance per component: `const measuredItemHeights = useRef(new Map())`.
- **R-CACHE-003** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount: `useEffect(() => { return () => { measureRefsRef.current.delete(componentId) } }, [componentId])`.
- **R-CACHE-004** SHOULD: Access cache within useCallback hooks for measurement operations to maintain O(1) lookup performance.
- **R-CACHE-005** SHOULD: Add cache size monitoring in development builds to detect potential memory leaks early.

### Verify

```bash
# Verify Map-based cache patterns are present
grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/

# Verify cache operations use componentId keys
grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/

# Run cache and measurement-related tests
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'
```

**Accept when:**
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded
- useRef initialization pattern is used for all Map-based caches
- useEffect cleanup functions properly delete cache entries on unmount

<enforcement>
Claude Code MUST NOT skip or defer verification. All virtual scroll components must conform to Map-based caching patterns before approval. Performance regression tests must pass with scroll interaction frame rates at or above 60fps threshold.
</enforcement>