# Adopt Map-Based Caching for Component Measurement State in Public API Components: Cached Measurement Data

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Public API components in packages/core expose virtualized and drag-drop interfaces (VirtualizedDropZone, DragDropContext) that require dynamic measurement and state coordination across component lifecycles
- Component measurement data (heights, refs, virtualizer handles) must be cached and retrieved efficiently during render cycles to support @tanstack/react-virtual and @dnd-kit integration patterns
- React hooks (useCallback, useMemo, useEffect) coordinate with Map-based caches to manage component-scoped state that persists across re-renders but remains isolated per component instance
- Demo applications consume these core components and use URLSearchParams.get() to retrieve configuration from query parameters, establishing a consistent pattern of key-value retrieval across the public API surface

## Problem Statement

Public API components require efficient, component-scoped caching mechanisms to store and retrieve measurement data, virtualizer handles, and configuration parameters without introducing prop-drilling or global state pollution, while maintaining compatibility with React's rendering model and third-party virtualization libraries.

## Decision

1. MUST: Cached measurement data MUST be cleaned up via .delete() when components unmount or identifiers change

## Policy Block

- MUST Cached measurement data MUST be cleaned up via .delete() when components unmount or identifiers change

In scope:
- Public API components exported from packages/core (VirtualizedDropZone, DragDropContext, Puck)
- Component measurement and virtualization state managed via @tanstack/react-virtual or @dnd-kit libraries
- Client-side configuration retrieval in demo applications using URLSearchParams
- React hook-coordinated cache operations within component lifecycle methods

Out of scope:
- Server-side state management or SSR-specific caching strategies
- Global application state managed by external state management libraries
- Persistent storage mechanisms (localStorage, IndexedDB, cookies)
- Cache invalidation strategies beyond component unmount cleanup

## Rationale

- Map-based caching provides O(1) lookup performance for component-scoped state while maintaining referential stability across React re-renders, as evidenced by measuredItemHeights.get/set patterns in VirtualizedDropZone
- The pattern separates transient UI state from application state, allowing public API components to manage their own measurement data without coupling to external state management solutions
- Consistent use of .get() retrieval across both Map instances (measuredItemHeights.get) and URLSearchParams (params.get) establishes a uniform key-value access pattern across the public API surface
- Integration with @tanstack/react-virtual and @dnd-kit requires component-local caches that can be synchronized with external virtualizer handles (rootVirtualizers.set/delete) while maintaining cleanup guarantees

## Consequences

Positive:
- Component measurement state remains isolated and garbage-collectable, preventing memory leaks in long-lived applications with dynamic component trees
- O(1) cache access performance supports high-frequency operations during scroll, drag, and virtualization events without introducing render bottlenecks
- Public API components remain framework-agnostic in their caching strategy, avoiding tight coupling to specific state management libraries
- Cleanup via .delete() ensures deterministic resource management when components unmount or identifiers change

Negative:
- Map-based caches require manual lifecycle management and cleanup logic, increasing implementation complexity compared to declarative state solutions
- Cache state is not serializable or inspectable via React DevTools, complicating debugging of measurement-related issues
- Pattern introduces multiple sources of truth (Map caches, React state, virtualizer state) that must be manually synchronized
- No built-in cache invalidation or staleness detection requires developers to implement custom cleanup strategies

## Alternatives

- Use React useState/useReducer for all component measurement state instead of Map-based caches (rejected)
  Rejected because: React state updates trigger re-renders on every measurement change, creating performance bottlenecks during high-frequency virtualization and drag events where measurements update faster than render cycles
  When valid: For low-frequency state updates or when React DevTools inspection is critical for debugging
- Adopt WeakMap for automatic garbage collection of component-scoped caches (rejected)
  Rejected because: WeakMap requires object keys rather than string identifiers (componentId), and the evidence shows string-based component identifiers are used throughout the codebase
  When valid: When component instances themselves (object references) serve as natural cache keys and automatic GC is required
- Integrate a dedicated caching library (e.g., lru-cache) for measurement state management (rejected)
  Rejected because: Adds external dependency overhead for simple key-value storage patterns that native Map already provides, and evidence shows no LRU eviction requirements
  When valid: When cache size limits, TTL expiration, or LRU eviction policies are required for bounded memory usage

## Risks

- Memory leaks if .delete() cleanup is not consistently implemented in all component unmount paths or when component identifiers change
  Mitigation: Establish linting rules to verify useEffect cleanup functions call .delete() for all Map-based caches, and implement integration tests that verify cache cleanup after component unmount
  Owner: Core Components Team
- Cache synchronization bugs when Map state diverges from React state or virtualizer state during concurrent updates
  Mitigation: Document cache update ordering requirements in component implementation guides, and add runtime assertions in development mode to detect state divergence
  Owner: Core Components Team
- Debugging difficulty due to Map state being invisible to React DevTools and standard state inspection tools
  Mitigation: Implement custom DevTools integration or debug logging utilities that expose Map cache contents during development, and document cache inspection patterns in troubleshooting guides
  Owner: Developer Experience Team

## Implementation Notes

- Initialize Map instances at component scope (not inside hooks) to ensure cache stability across re-renders: const measuredItemHeights = new Map()
- Coordinate cache operations with React hooks: use useEffect for cleanup (.delete()), useCallback for cache-dependent callbacks, and useMemo for derived cache values
- Implement cleanup in useEffect return functions: return () => { measuredItemHeights.delete(componentId); measureRefsRef.current.delete(componentId); }
- For URLSearchParams caching in client components, guard with typeof window checks: const params = typeof window === 'undefined' ? new URLSearchParams() : new URL(window.location.href).searchParams
- Document cache key formats and lifecycle expectations in component API documentation to ensure consistent usage across public API consumers

## Continuation Context


Verify commands:
- grep -r '\.get(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers|params)\.get'
- grep -r '\.set(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.set'
- grep -r '\.delete(' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)\.delete'
- grep -r 'params\.get' apps/demo --include='*.tsx' | grep 'URLSearchParams'

Accept when:
- All public API components in packages/core that manage measurement state use Map instances with .get(), .set(), and .delete() operations
- Every .set() operation on component-scoped Map caches has a corresponding .delete() in a useEffect cleanup function or component unmount path
- Demo applications consistently use URLSearchParams.get() for query parameter retrieval in client-side contexts with typeof window guards

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for .get/.set/.delete patterns in packages/core/components
- Verified by: Code review checklist requiring verification of Map cleanup in useEffect return functions
- Verified by: Integration tests validating cache cleanup after component unmount in VirtualizedDropZone and DragDropContext
- Violation handling: CI pipeline fails if Map-based cache operations are detected without corresponding cleanup logic
- Violation handling: Code review blocks merge if new Map caches are introduced without documented lifecycle management
- Violation handling: Runtime warnings in development mode when cache size grows beyond expected thresholds, indicating potential cleanup failures
- Exception process: Document exception rationale in component implementation comments explaining why cleanup is deferred or unnecessary
- Exception process: Obtain approval from Core Components Team lead for any Map-based cache without explicit .delete() cleanup
- Exception process: Add exception to linting configuration with inline comment explaining architectural justification