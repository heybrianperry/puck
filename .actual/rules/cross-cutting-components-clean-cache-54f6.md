# Use Map-Based Measurement Caching for Virtual Scroll Component State: Components Clean Cache

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components/DropZone/VirtualizedDropZone.tsx and packages/core/components/DragDropContext/index.tsx, and any components using @tanstack/react-virtual or @dnd-kit/react libraries that require dynamic measurement caching.

### Rules

- **R-CACHE-001** SHOULD: Components SHOULD clean up cache entries via delete operations when components unmount or are removed from the virtual list.
- **R-CACHE-002** MUST: Initialize Map caches using useRef to ensure a single instance per component: `const measuredItemHeights = useRef(new Map())`.
- **R-CACHE-003** MUST: Access cache within useCallback hooks for measurement operations to maintain referential stability.
- **R-CACHE-004** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount: `useEffect(() => { return () => { measureRefsRef.current.delete(componentId) } }, [componentId])`.
- **R-CACHE-005** SHOULD: Consider adding cache size monitoring in development builds to detect potential memory leaks early.

### Verify

```bash
# Verify Map-based cache patterns are present
grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/

# Verify cache operations use componentId keys
grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/

# Run tests for cache and measurement behavior
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'
```

**Accept when:**
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded
- useEffect cleanup functions explicitly call delete operations on cache entries
- No reactive state (useState) is used for storing measurement data

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing or implementing virtual scroll components with measurement caching requirements.
</enforcement>