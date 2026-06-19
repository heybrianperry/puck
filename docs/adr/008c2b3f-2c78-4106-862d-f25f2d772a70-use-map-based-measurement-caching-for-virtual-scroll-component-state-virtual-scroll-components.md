# Use Map-Based Measurement Caching for Virtual Scroll Component State: Virtual Scroll Components

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Virtual scrolling components in packages/core/components/DropZone/VirtualizedDropZone.tsx and packages/core/components/DragDropContext/index.tsx require efficient storage and retrieval of dynamic item measurements during render cycles
- React component lifecycle demands that measurement data persist across re-renders without triggering unnecessary updates, necessitating a cache layer outside the reactive state system
- The @tanstack/react-virtual and @dnd-kit/react libraries coordinate drag-drop interactions with virtualized rendering, requiring fast lookup of component dimensions by componentId keys
- Map data structures provide O(1) access patterns for get/set/delete operations on measurement data (measuredItemHeights) and reference tracking (measureRefsRef.current), supporting real-time UI interactions

## Problem Statement

Virtual scrolling and drag-drop components require a performant cache layer to store and retrieve dynamic item measurements and references without coupling to React's reactive state system, which would cause excessive re-renders and degrade interaction performance during scroll and drag operations.

## Decision

1. MUST: Virtual scroll components MUST use Map data structures for caching item measurements indexed by componentId

## Policy Block

- MUST Virtual scroll components MUST use Map data structures for caching item measurements indexed by componentId

In scope:
- Virtual scroll components using @tanstack/react-virtual
- Drag-drop contexts using @dnd-kit/react and @dnd-kit/dom
- Components in packages/core/components requiring dynamic measurement caching
- Item height tracking and measurement reference storage

Out of scope:
- Static component layouts without virtualization
- Server-side rendering contexts where measurements are unavailable
- Components with fixed-height items that do not require measurement
- Global application state managed by store systems

## Rationale

- Map data structures provide O(1) lookup performance for componentId-keyed measurements, essential for maintaining 60fps during scroll and drag interactions
- Storing caches in React refs prevents coupling to reactive state, avoiding re-render cascades that would degrade performance in virtualized lists with hundreds of items
- The pattern is observed in 2 files with 87.10% confidence, demonstrating consistent application across VirtualizedDropZone and DragDropContext components that coordinate with @tanstack/react-virtual and @dnd-kit libraries
- Explicit delete operations (measureRefsRef.current.delete, rootVirtualizers.delete) indicate intentional memory management for dynamic component lifecycles

## Consequences

Positive:
- O(1) access time for measurement lookups during scroll and drag operations maintains interaction responsiveness
- Decoupling cache from React state prevents unnecessary re-renders and improves rendering performance
- Explicit cleanup via delete operations prevents memory leaks in long-lived virtualized lists
- Pattern integrates cleanly with @tanstack/react-virtual and @dnd-kit libraries' measurement APIs

Negative:
- Map-based caches stored in refs are invisible to React DevTools, complicating debugging of measurement state
- Manual memory management via delete operations increases implementation complexity and risk of memory leaks if cleanup is missed
- Cache invalidation logic must be manually coordinated with component lifecycle, lacking automatic garbage collection
- Pattern creates imperative data access within declarative React components, mixing programming paradigms

## Alternatives

- Store measurements in React useState with componentId-keyed objects (rejected)
  Rejected because: Would trigger re-renders on every measurement update, causing performance degradation in virtualized lists with frequent scroll events
  When valid: Acceptable for small lists (<50 items) where re-render cost is negligible
- Use WeakMap for automatic garbage collection of unmounted component measurements (rejected)
  Rejected because: WeakMap keys must be objects, but componentId appears to be string-based in the evidence (componentId parameter), making WeakMap incompatible
  When valid: Valid if componentId were refactored to use object references instead of string identifiers
- Delegate all measurement caching to @tanstack/react-virtual internal state (rejected)
  Rejected because: Evidence shows custom measurement tracking (measuredItemHeights, measureRefsRef) alongside library usage, suggesting library's internal caching is insufficient for this use case
  When valid: Valid for simpler virtualization scenarios without drag-drop coordination requirements

## Risks

- Memory leaks if delete operations are not called when components unmount or are removed from virtual list
  Mitigation: Implement useEffect cleanup functions that call delete on cache entries; add integration tests verifying cache size remains bounded during mount/unmount cycles
  Owner: engineering team
- Cache inconsistency if componentId keys collide or are reused across different component instances
  Mitigation: Ensure componentId generation produces unique, stable identifiers; document componentId requirements in component interfaces
  Owner: engineering team
- Debugging difficulty due to cache state being invisible to React DevTools
  Mitigation: Add custom DevTools integration or logging utilities that expose Map cache contents; document cache inspection techniques in developer guides
  Owner: engineering team

## Implementation Notes

- Initialize Map caches using useRef to ensure single instance per component: const measuredItemHeights = useRef(new Map())
- Access cache within useCallback hooks for measurement operations: useCallback((componentId, height) => { measuredItemHeights.current.set(componentId, height) }, [])
- Implement cleanup in useEffect return functions: useEffect(() => { return () => { measureRefsRef.current.delete(componentId) } }, [componentId])
- Consider adding cache size monitoring in development builds to detect potential memory leaks early

## Continuation Context


Verify commands:
- grep -r 'measuredItemHeights.*Map\|measureRefsRef.*Map\|rootVirtualizers.*Map' packages/core/components/
- grep -r '\.get(componentId)\|\.set(componentId\|\.delete(componentId)' packages/core/components/DropZone/ packages/core/components/DragDropContext/
- npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cache|measurement'

Accept when:
- All virtual scroll components use Map data structures stored in refs for measurement caching
- Cache operations (get/set/delete) are present for componentId-keyed measurements in VirtualizedDropZone and DragDropContext
- Tests verify cache cleanup occurs on component unmount and cache size remains bounded

## Enforcement

- Verified by: Code review checklist verifying Map-based cache pattern in new virtual scroll components
- Verified by: Automated grep checks in CI pipeline detecting measurement cache patterns
- Verified by: Integration tests validating cache behavior during mount/unmount cycles
- Violation handling: Code review rejection if virtual scroll components use reactive state for measurement caching
- Violation handling: CI pipeline warnings if Map cache patterns are missing in components importing @tanstack/react-virtual
- Violation handling: Performance regression tests fail if scroll interaction frame rates drop below 60fps threshold
- Exception process: Document performance requirements and justify alternative approach in component documentation
- Exception process: Obtain approval from architecture review board for non-Map caching strategies
- Exception process: Provide benchmark data demonstrating equivalent or superior performance characteristics