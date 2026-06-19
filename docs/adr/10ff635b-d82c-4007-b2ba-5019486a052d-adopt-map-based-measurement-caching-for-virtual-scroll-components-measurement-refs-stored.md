# Adopt Map-Based Measurement Caching for Virtual Scroll Components: Measurement Refs Stored

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Virtual scrolling components in the packages/core module require efficient measurement caching to maintain performance when rendering large lists with dynamic item heights
- The VirtualizedDropZone and DragDropContext components coordinate drag-and-drop interactions with @dnd-kit/react and @tanstack/react-virtual, requiring persistent measurement state across re-renders
- Component-level measurement data (heights, refs) must be stored and retrieved using component IDs as keys to support dynamic list operations including insertion, deletion, and reordering
- The integration pattern emerged from the need to bridge stateful UI interactions (useCallback, useMemo, useEffect) with external virtualizer handles and measurement tracking

## Problem Statement

Virtual scroll components with drag-and-drop capabilities require a consistent mechanism to cache and retrieve item measurements across render cycles, coordinate with external virtualizer libraries, and maintain measurement state during dynamic list mutations without causing performance degradation or measurement inconsistencies.

## Decision

1. MUST: Measurement refs MUST be stored in a separate Map structure (measureRefsRef.current) to maintain reference stability across renders

## Policy Block

- MUST Measurement refs MUST be stored in a separate Map structure (measureRefsRef.current) to maintain reference stability across renders

In scope:
- Virtual scroll components in packages/core/components that integrate with @tanstack/react-virtual
- Drag-and-drop contexts that coordinate with @dnd-kit/react and require measurement tracking
- Components that manage dynamic item heights and require measurement caching across re-renders
- Multi-zone virtualizer coordination requiring handle registration and lookup

Out of scope:
- Static list components with fixed item heights that do not require measurement caching
- Non-virtualized components regardless of list size
- Measurement caching for non-React frameworks or libraries
- Server-side rendering contexts where measurement caching is not applicable

Exceptions:
- EXC-001: Component uses a third-party virtualization library that provides its own measurement caching mechanism incompatible with Map-based storage

## Rationale

- Map data structures provide O(1) lookup, insertion, and deletion performance for component ID-based measurement retrieval, which is critical for maintaining 60fps scrolling performance in large lists
- The pattern is evidenced in 2 files (VirtualizedDropZone.tsx, DragDropContext/index.tsx) with 87.10% confidence, showing consistent adoption of measuredItemHeights.get/set/delete and measureRefsRef.current.get/set/delete operations
- Separation of measurement data (heights) and measurement refs into distinct Map structures maintains React's ref stability requirements while enabling efficient cache invalidation
- The integration with @tanstack/react-virtual and @dnd-kit/react requires a coordination layer that bridges external virtualizer handles with internal measurement state, achieved through Map-based registration (rootVirtualizers.set/delete)

## Consequences

Positive:
- Consistent O(1) performance for measurement lookups regardless of list size, preventing performance degradation in large lists
- Clear separation of concerns between measurement data and measurement refs enables independent cache invalidation strategies
- Map-based storage provides built-in key existence checking and iteration capabilities useful for debugging and state inspection
- Pattern enables coordination between multiple virtualizer instances through centralized handle registration

Negative:
- Map structures require manual memory management through explicit delete operations, creating potential for memory leaks if cleanup is missed
- Additional complexity in component lifecycle management to ensure cache entries are synchronized with component mount/unmount cycles
- Debugging measurement cache state requires understanding Map data structure inspection techniques rather than simple object property inspection
- Pattern introduces coupling between component IDs and cache keys, requiring stable ID generation strategies

## Alternatives

- Use plain JavaScript objects with component IDs as string keys for measurement caching (rejected)
  Rejected because: Object property access has prototype chain overhead and lacks built-in size tracking; Map provides cleaner semantics for key-value caching with non-string keys and better iteration performance
  When valid: Acceptable for components with fewer than 100 items where prototype chain overhead is negligible and serialization to JSON for debugging is required
- Delegate all measurement caching to @tanstack/react-virtual internal state without explicit Map-based layer (rejected)
  Rejected because: Drag-and-drop coordination with @dnd-kit/react requires access to measurements outside the virtualizer lifecycle; explicit cache layer enables cross-library state sharing
  When valid: Valid for pure virtualization scenarios without drag-and-drop or other external integrations requiring measurement access
- Use WeakMap for automatic garbage collection of measurement cache entries (rejected)
  Rejected because: WeakMap requires object keys but component IDs are strings; automatic GC prevents explicit cache invalidation control needed for coordinating with virtualizer updates
  When valid: Could be reconsidered if component IDs are refactored to object references and automatic cleanup is proven safe for virtualizer coordination

## Risks

- Memory leaks from orphaned cache entries if components unmount without calling delete operations on measurement Maps
  Mitigation: Implement useEffect cleanup functions that explicitly delete cache entries on component unmount; add development-mode warnings for cache size growth
  Owner: Core UI Engineering Team
- Cache invalidation bugs where stale measurements cause incorrect scroll positioning or layout thrashing
  Mitigation: Establish clear cache invalidation rules tied to component lifecycle events; implement verification that measurements are refreshed when item heights change
  Owner: Core UI Engineering Team
- Performance degradation if Map iteration is used in hot render paths instead of targeted get operations
  Mitigation: Code review guidelines prohibiting Map.forEach or Array.from(map) in render methods; use targeted get/set operations only
  Owner: Architecture Review Team

## Implementation Notes

- Initialize measurement Maps using useRef to maintain reference stability across renders: const measuredItemHeights = useRef(new Map())
- Pair every set operation with a corresponding delete in useEffect cleanup: useEffect(() => { map.set(id, value); return () => map.delete(id); }, [id])
- Use Map.has() to check for cache hits before calling get() to distinguish between cached undefined values and cache misses
- For multi-zone coordination, use compound keys (e.g., zoneCompound) that uniquely identify virtualizer instances across the component tree
- Consider exposing cache size metrics (map.size) in development mode to detect memory leaks during testing

## Continuation Context


Verify commands:
- grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ | wc -l
- grep -r 'measureRefsRef\.current\.\(get\|set\|delete\)' packages/core/components/ | wc -l
- grep -r 'new Map()' packages/core/components/ | grep -E '(measured|cache|virtualizer)' | wc -l

Accept when:
- All virtual scroll components use Map.get/set/delete operations for measurement caching with component IDs as keys
- Each Map.set operation has a corresponding Map.delete in useEffect cleanup or component unmount handler
- No usage of plain objects or arrays for measurement caching in virtualized components that integrate with @tanstack/react-virtual

## Enforcement

- Verified by: Code review checklist requiring Map-based caching for all new virtual scroll components
- Verified by: ESLint custom rule detecting measurement caching patterns and flagging non-Map implementations
- Verified by: Integration tests verifying measurement cache cleanup on component unmount
- Violation handling: Pull requests introducing non-Map measurement caching in virtual scroll components are blocked until refactored
- Violation handling: Existing violations are tracked in technical debt backlog with priority based on component usage frequency
- Violation handling: Performance regression tests flag components with measurement cache memory leaks
- Exception process: Submit exception request to Architecture Review Team with justification for alternative caching strategy
- Exception process: Provide performance benchmarks comparing alternative approach to Map-based caching
- Exception process: Document approved exceptions in component README with rationale and monitoring plan