# Use Map-Based Measurement Caching for Virtual Scroll Component State: Virtual Scroll Components

These rules are ALWAYS ACTIVE for virtual scroll components in packages/core/components/DropZone/VirtualizedDropZone.tsx and packages/core/components/DragDropContext/index.tsx, and any components using @tanstack/react-virtual or @dnd-kit/react libraries that require dynamic item measurement caching.

### Rules

- **R-VSCROLL-001** MUST: Virtual scroll components MUST use Map data structures for caching item measurements indexed by componentId.
- **R-VSCROLL-002** MUST: Initialize Map caches using useRef to ensure a single instance per component: `const measuredItemHeights = useRef(new Map())`.
- **R-VSCROLL-003** MUST: Access cache within useCallback hooks for measurement operations to maintain referential stability.
- **R-VSCROLL-004** MUST: Implement cleanup in useEffect return functions to delete cache entries when components unmount: `useEffect(() => { return () => { measureRefsRef.current.delete(componentId) } }, [componentId])`.
- **R-VSCROLL-005** SHOULD: Add cache size monitoring in development builds to detect potential memory leaks early.
- **R-VSCROLL-006** SHOULD: Ensure componentId generation produces unique, stable identifiers and document componentId requirements in component interfaces.

### Verify

```bash
# Verify Map-based cache patterns exist in virtual scroll components
grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/

# Verify cache operations (get/set/delete) are present for componentId-keyed measurements
grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/

# Run tests for virtual scroll components and measurement caching
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'
```

**Accept when:**
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded
- No reactive state (useState) is used for storing measurement data in virtual scroll components
- useRef is used to initialize all measurement cache Maps
- useEffect cleanup functions properly delete cache entries on unmount

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based cache patterns in virtual scroll components. All R-VSCROLL rules are mandatory for components using @tanstack/react-virtual or @dnd-kit/react.
</enforcement>