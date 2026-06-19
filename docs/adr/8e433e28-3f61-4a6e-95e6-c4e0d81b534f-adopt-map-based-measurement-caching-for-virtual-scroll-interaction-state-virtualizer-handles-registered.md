# Adopt Map-Based Measurement Caching for Virtual Scroll Interaction State: Virtualizer Handles Registered

Status: proposed
Date: 2024-01-20
Deciders: Detection Pipeline (automated)

## Context

- Virtual scrolling components require dynamic measurement of item heights to calculate viewport positioning and render only visible items efficiently
- React component re-renders and interaction callbacks (useCallback, useEffect, useMemo) need access to cached measurement data without triggering additional renders
- The codebase uses @tanstack/react-virtual and @dnd-kit/react for virtualized lists and drag-drop interactions, requiring coordination between measurement state and interaction handlers
- Multiple virtualized zones and drop zones must maintain independent measurement caches while sharing a common store pattern for state management

## Problem Statement

Virtual scroll components with drag-and-drop interactions require a performant mechanism to cache and retrieve item measurements across render cycles without causing unnecessary re-renders, while maintaining measurement isolation between different virtualized zones and supporting dynamic updates during user interactions.

## Decision

1. SHOULD: Virtualizer handles SHOULD be registered in a root-level Map structure to coordinate multiple virtualized zones

## Policy Block

- SHOULD Virtualizer handles SHOULD be registered in a root-level Map structure to coordinate multiple virtualized zones

In scope:
- Virtual scroll components using @tanstack/react-virtual
- Drag-and-drop zones using @dnd-kit/react or @dnd-kit/abstract
- Components requiring dynamic item height measurement
- Interaction handlers (useCallback, useEffect) accessing measurement data
- Multi-zone virtualized layouts requiring independent caches

Out of scope:
- Static lists without virtualization
- Components with fixed-height items that do not require measurement
- Non-React UI frameworks
- Server-side rendering of list components
- Simple state management without performance optimization requirements

## Rationale

- Map data structures provide O(1) lookup performance for measurement retrieval by component ID, critical for maintaining 60fps scroll performance in virtualized lists
- Storing measurement caches in React refs prevents cache updates from triggering re-renders, isolating measurement operations from the React render cycle
- The pattern is observed across 3 files with 85.83% confidence, demonstrating consistent adoption in VirtualizedDropZone, DragDropContext, and client components
- Integration with @tanstack/react-virtual and @dnd-kit libraries requires a cache layer that can be accessed synchronously during scroll and drag events without async state updates

## Consequences

Positive:
- Eliminates unnecessary re-renders caused by measurement updates, improving scroll performance in large virtualized lists
- Provides O(1) lookup time for item measurements during scroll calculations and drag-drop operations
- Enables independent measurement caching for multiple virtualized zones without state conflicts
- Maintains compatibility with React's concurrent rendering model by isolating imperative measurement operations in refs

Negative:
- Increases memory usage proportional to the number of measured items across all virtualized zones
- Requires manual cache cleanup logic to prevent memory leaks when components unmount
- Adds complexity to component lifecycle management with explicit Map operations (get, set, delete)
- Creates potential for stale measurement data if cache invalidation is not properly implemented

## Alternatives

- Store measurements in React state using useState or useReducer (rejected)
  Rejected because: State updates trigger re-renders on every measurement change, causing performance degradation in virtualized lists with frequent scroll events
  When valid: Acceptable for small lists (<50 items) where re-render cost is negligible
- Use a global singleton cache outside React component tree (rejected)
  Rejected because: Breaks React's component isolation model and complicates server-side rendering and testing with shared mutable state
  When valid: Valid for applications with a single virtualized list instance and no SSR requirements
- Rely on CSS-based fixed heights without dynamic measurement (rejected)
  Rejected because: Does not support variable-height content or dynamic content that requires measurement after render
  When valid: Appropriate for lists with uniform, known item heights that do not change

## Risks

- Memory leaks if measurement caches are not properly cleaned up when components unmount or items are removed
  Mitigation: Implement cleanup logic in useEffect return functions and component unmount handlers to delete cache entries
  Owner: engineering team
- Stale measurement data if items resize without triggering cache invalidation
  Mitigation: Implement ResizeObserver or mutation observers to detect size changes and update cache entries
  Owner: engineering team
- Debugging difficulty due to measurement state stored in refs rather than React DevTools-visible state
  Mitigation: Add logging or custom DevTools integration to expose measurement cache contents during development
  Owner: engineering team

## Implementation Notes

- Use useRef to create Map instances for measurement caches: `const measuredItemHeights = useRef(new Map())`
- Access cache within callbacks using `.current`: `measuredItemHeights.current.get(componentId)` and `measuredItemHeights.current.set(componentId, height)`
- Implement cleanup in useEffect return: `return () => { measuredItemHeights.current.delete(componentId); }`
- For multi-zone coordination, maintain a root-level Map of virtualizer handles keyed by zone compound identifiers
- Consider using WeakMap if component IDs are object references to enable automatic garbage collection

## Continuation Context


Verify commands:
- grep -r 'measuredItemHeights\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'
- grep -r 'useRef.*Map\|Map.*useRef' packages/core/components/ --include='*.tsx' --include='*.ts'
- grep -r 'rootVirtualizers\.\(get\|set\|delete\)' packages/core/components/ --include='*.tsx' --include='*.ts'

Accept when:
- Virtualized components use Map data structures stored in React refs for measurement caching
- Measurement cache operations (get, set, delete) are present in interaction hooks (useCallback, useEffect, useMemo)
- Cleanup logic exists to delete cache entries on component unmount or item removal

## Enforcement

- Verified by: Code review checking for Map-based measurement caching in virtualized components
- Verified by: Automated grep patterns in CI pipeline detecting measurement cache operations
- Verified by: Performance testing validating scroll performance meets 60fps threshold
- Violation handling: Code review feedback requesting refactor to Map-based caching for virtualized components
- Violation handling: Performance regression tests failing if scroll performance degrades below threshold
- Violation handling: Architecture review required for alternative caching approaches
- Exception process: Document justification for alternative approach in component documentation
- Exception process: Obtain approval from frontend architecture team for non-Map caching strategies
- Exception process: Provide performance benchmarks demonstrating equivalent or better performance