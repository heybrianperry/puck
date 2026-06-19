# Use Map-Based Measurement Caching for Virtual Scroll Component State: Measurement Caches Support

These rules are ALWAYS ACTIVE for virtual scrolling and drag-drop components in packages/core/components requiring dynamic measurement caching, including VirtualizedDropZone and DragDropContext components that coordinate with @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-MEAS-001** MUST: Measurement caches MUST support get, set, and delete operations for component height data and measurement references.
- **R-MEAS-002** MUST: Initialize Map caches using useRef to ensure a single instance per component: `const measuredItemHeights = useRef(new Map())`.
- **R-MEAS-003** MUST: Access cache within useCallback hooks for measurement operations to maintain referential stability.
- **R-MEAS-004** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount or are removed from virtual lists.
- **R-MEAS-005** SHOULD: Add cache size monitoring in development builds to detect potential memory leaks early.
- **R-MEAS-006** SHOULD: Ensure componentId generation produces unique, stable identifiers and document componentId requirements in component interfaces.

### Verify

```bash
# Verify Map-based cache patterns exist in virtual scroll components
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
- useRef initialization pattern is used for all measurement cache declarations
- useEffect cleanup functions properly delete cache entries on unmount

<enforcement>
Claude Code MUST NOT skip or defer verification. All virtual scroll components must be audited for Map-based cache compliance before approval. Performance regression tests must pass to confirm 60fps interaction responsiveness is maintained.
</enforcement>