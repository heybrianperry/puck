# Use Map-Based Measurement Caching for Virtual Scroll Component State: Additional Map Based

These rules are ALWAYS ACTIVE for virtual scroll components using @tanstack/react-virtual, drag-drop contexts using @dnd-kit/react and @dnd-kit/dom, and components in packages/core/components requiring dynamic measurement caching.

### Rules

- **R-VSCACHE-001** MAY: Additional Map-based caches MAY be used for related state such as virtualizer handles (rootVirtualizers) or index tracking (nextPinnedIndexes).
- **R-VSCACHE-002** MUST: Initialize Map caches using useRef to ensure single instance per component: `const measuredItemHeights = useRef(new Map())`.
- **R-VSCACHE-003** MUST: Access cache within useCallback hooks for measurement operations to maintain referential stability.
- **R-VSCACHE-004** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount or are removed from virtual list.
- **R-VSCACHE-005** SHOULD: Add cache size monitoring in development builds to detect potential memory leaks early.
- **R-VSCACHE-006** MUST: Ensure componentId generation produces unique, stable identifiers and document componentId requirements in component interfaces.

### Verify

```bash
# Detect Map-based cache patterns in virtual scroll components
grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/

# Verify cache operations (get/set/delete) are present for componentId-keyed measurements
grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/

# Run tests for virtual scroll and drag-drop components
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'
```

**Accept when:**
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded
- useRef initialization pattern is used for all Map-based caches
- useCallback hooks wrap measurement operations
- useEffect cleanup functions call delete on cache entries

<enforcement>
Clause Code MUST NOT skip or defer verification. All virtual scroll components must conform to Map-based caching patterns. Code review rejection occurs if virtual scroll components use reactive state for measurement caching. CI pipeline warnings trigger if Map cache patterns are missing in components importing @tanstack/react-virtual. Performance regression tests fail if scroll interaction frame rates drop below 60fps threshold.
</enforcement>