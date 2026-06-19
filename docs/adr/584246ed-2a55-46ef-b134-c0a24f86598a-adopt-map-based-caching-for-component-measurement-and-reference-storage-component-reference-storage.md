# Adopt Map-Based Caching for Component Measurement and Reference Storage: Component Reference Storage

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React components require efficient storage and retrieval of dynamically measured UI element dimensions for virtualization and layout calculations
- Component references and height measurements must persist across render cycles without triggering unnecessary re-renders
- URL query parameters need to be parsed and cached for runtime configuration decisions in client-side applications
- The codebase uses @tanstack/react-virtual and @dnd-kit libraries which require imperative access to measurement data outside the React render cycle

## Problem Statement

React applications with virtualized lists, drag-and-drop interactions, and dynamic layouts require a performant mechanism to store and retrieve component measurements, DOM references, and configuration parameters without causing render cascades or memory leaks. Traditional state management approaches trigger re-renders on every update, while Map-based caching provides O(1) access with imperative mutation semantics suitable for measurement tracking.

## Decision

1. MUST: Component reference storage MUST use Map.get() and Map.set() operations for retrieval and storage of DOM measurement references

## Policy Block

- MUST Component reference storage MUST use Map.get() and Map.set() operations for retrieval and storage of DOM measurement references

In scope:
- Component measurement caching in virtualized lists
- DOM reference storage for drag-and-drop interactions
- URL query parameter parsing and caching
- Virtualizer handle registration and cleanup

Out of scope:
- Server-side data caching
- HTTP response caching
- Redux or global state management
- LocalStorage or SessionStorage persistence

## Rationale

- Map data structures provide O(1) lookup and insertion performance critical for real-time measurement tracking during scroll and drag operations
- Imperative Map mutations avoid triggering React re-renders, preventing performance degradation in virtualized lists with hundreds of items
- The evidence shows consistent usage across 4 files with measuredItemHeights.get/set/delete and measureRefsRef.current.get/set/delete patterns
- URLSearchParams.get() provides a standard browser API for query parameter caching without additional dependencies

## Consequences

Positive:
- O(1) performance for measurement lookups during scroll and virtualization calculations
- No unnecessary re-renders when measurement data is updated imperatively
- Clean memory management through explicit Map.delete() calls on component unmount
- Standard browser APIs reduce dependency footprint for query parameter handling

Negative:
- Imperative Map mutations bypass React's declarative model, making data flow harder to trace
- Map contents are not visible to React DevTools, complicating debugging
- Manual cleanup responsibility increases risk of memory leaks if delete() calls are missed
- No built-in serialization support for Map structures limits persistence options

## Alternatives

- Use React useState for measurement storage (rejected)
  Rejected because: setState triggers re-renders on every measurement update, causing severe performance degradation in virtualized lists with frequent scroll events
  When valid: Only valid for small lists with infrequent measurement updates
- Use WeakMap for automatic garbage collection (rejected)
  Rejected because: WeakMap requires object keys while component IDs are strings, and lacks iteration capabilities needed for bulk operations
  When valid: Valid when keys are objects and iteration is not required
- Use plain JavaScript objects for caching (deferred)
  Rejected because: Objects lack explicit get/set/delete semantics and have prototype chain lookup overhead
  When valid: Valid for simple key-value caching without frequent deletions

## Risks

- Memory leaks if Map.delete() cleanup is not called on component unmount
  Mitigation: Enforce useEffect cleanup functions that call Map.delete() for all registered component IDs
  Owner: engineering team
- Race conditions when multiple components update the same Map concurrently
  Mitigation: Use component-scoped identifiers as keys to ensure isolation between components
  Owner: engineering team
- Debugging difficulty due to Map contents being invisible to React DevTools
  Mitigation: Add development-mode logging for Map operations and provide custom DevTools integration
  Owner: engineering team

## Implementation Notes

- Store Map instances in useRef to prevent re-creation on every render: const mapRef = useRef(new Map())
- Always pair Map.set() calls with corresponding Map.delete() calls in useEffect cleanup functions
- Use componentId or similar unique identifiers as Map keys to prevent collisions
- For URLSearchParams caching, instantiate once and reuse: const params = new URL(window.location.href).searchParams

## Continuation Context


Verify commands:
- grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/
- grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/
- grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

Accept when:
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching

## Enforcement

- Verified by: Code review checklist verifying Map cleanup in useEffect returns
- Verified by: ESLint custom rule detecting Map.set without corresponding delete
- Verified by: Runtime memory profiling in CI to detect Map growth patterns
- Violation handling: CI build fails if Map operations are detected without cleanup functions
- Violation handling: Code review blocks merge if Map usage does not follow ref storage pattern
- Violation handling: Runtime warnings in development mode when Map size exceeds thresholds
- Exception process: Document exception rationale in code comments explaining why cleanup is not needed
- Exception process: Obtain approval from tech lead for Map usage without ref storage
- Exception process: Add eslint-disable comment with ticket reference for tracking