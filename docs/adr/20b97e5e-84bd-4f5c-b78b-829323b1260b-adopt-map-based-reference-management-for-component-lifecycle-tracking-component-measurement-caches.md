# Adopt Map-Based Reference Management for Component Lifecycle Tracking: Component Measurement Caches

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages virtualized UI components with dynamic lifecycle requirements, requiring efficient lookup and cleanup of component-specific state across render cycles
- React components in packages/core/components/DropZone/VirtualizedDropZone.tsx and packages/core/components/DragDropContext/index.tsx coordinate drag-drop interactions with @dnd-kit/react and @tanstack/react-virtual libraries
- Component measurement data (measuredItemHeights) and DOM references (measureRefsRef) require keyed access by componentId to support virtualization performance constraints
- The store architecture (../../store) necessitates subscription-based state synchronization across zone and area depth indexes, requiring efficient registration and deregistration patterns

## Problem Statement

Without a consistent pattern for managing component-scoped references and measurements across lifecycle boundaries, the system risks memory leaks from unreleased references, inconsistent state during component unmounting, and performance degradation from inefficient lookups in virtualized rendering contexts where hundreds of items may be measured and tracked simultaneously.

## Decision

1. SHOULD: Component measurement caches (measuredItemHeights) SHOULD use Map structures to support dynamic resizing and re-measurement without array reindexing overhead

## Policy Block

- SHOULD Component measurement caches (measuredItemHeights) SHOULD use Map structures to support dynamic resizing and re-measurement without array reindexing overhead

In scope:
- React components managing virtualized lists or grids with dynamic item counts
- Drag-and-drop contexts requiring per-zone or per-component state tracking
- Component measurement and layout systems using @tanstack/react-virtual or similar virtualization libraries
- Store subscription patterns where components register/unregister handlers keyed by identifier

Out of scope:
- Simple component state managed by useState or useReducer without cross-component coordination
- Global application state managed by Redux, Zustand, or similar state management libraries
- Server-side data caching or persistence layers
- Static component hierarchies without dynamic mounting/unmounting

Exceptions:
- EXC-001: Component identifiers are guaranteed to be sequential integers starting from 0 with no gaps, and array indexing provides equivalent performance

## Rationale

- The evidence shows consistent use of Map.get(), Map.set(), and Map.delete() operations across VirtualizedDropZone.tsx and DragDropContext/index.tsx, indicating an established pattern for component-keyed state management
- Virtualization libraries like @tanstack/react-virtual require efficient measurement caching where Map structures provide O(1) access without the reindexing overhead of array-based approaches
- The explicit .delete() calls in cleanup paths (measureRefsRef.current.delete(componentId), rootVirtualizers.delete(zoneCompound)) demonstrate intentional memory management to prevent reference accumulation
- The pattern supports the store subscription model where components register handlers by identifier and must cleanly unregister to avoid stale subscriptions

## Consequences

Positive:
- O(1) lookup, insertion, and deletion performance for component state regardless of total component count
- Explicit cleanup semantics reduce memory leak risk by making reference lifecycle visible in code
- Map keys support arbitrary identifier types (strings, numbers, compound keys) without array index constraints
- Pattern scales naturally to hundreds or thousands of virtualized items without performance degradation

Negative:
- Map structures consume more memory per entry than arrays for simple sequential integer keys
- Developers must remember to implement cleanup logic; missing .delete() calls create silent memory leaks
- Debugging Map contents requires explicit iteration or browser DevTools inspection rather than simple array logging
- Pattern introduces cognitive overhead for developers unfamiliar with Map API compared to array indexing

## Alternatives

- Use plain JavaScript objects with string keys for component state storage (rejected)
  Rejected because: Object property access requires string coercion for non-string keys, lacks explicit .delete() semantics (delete operator has performance implications), and does not provide size() or iteration guarantees
  When valid: When all component identifiers are guaranteed to be strings and cleanup is managed through object replacement rather than property deletion
- Use array indexing with componentId as array index (rejected)
  Rejected because: Requires sequential integer identifiers starting from 0, creates sparse arrays with gaps when components unmount, and forces array resizing/reindexing when component order changes
  When valid: When component identifiers are guaranteed sequential integers with no gaps and component count is bounded to small values (<100)
- Use WeakMap for automatic garbage collection of component references (deferred)
  Rejected because: WeakMap keys must be objects (not primitive componentId strings/numbers), lacks .size() and iteration capabilities needed for debugging, and automatic cleanup timing is non-deterministic
  When valid: When component references are already object-based and deterministic cleanup timing is not required for correctness

## Risks

- Developers may forget to implement Map.delete() in cleanup paths, causing memory leaks that accumulate over component mount/unmount cycles
  Mitigation: Establish linting rules or code review checklists requiring cleanup logic for all Map.set() operations; implement memory profiling tests that detect reference accumulation
  Owner: Engineering team
- Map key collisions may occur if componentId generation is not globally unique across component types or instances
  Mitigation: Document componentId generation contracts; use compound keys (e.g., zoneCompound) when multiple namespaces exist; implement runtime assertions for duplicate key detection in development builds
  Owner: Engineering team
- Map structures may be over-applied to simple cases where useState would suffice, increasing code complexity unnecessarily
  Mitigation: Define clear policy scope (in-scope: cross-component coordination, virtualization; out-of-scope: simple local state); provide decision tree in documentation
  Owner: Architecture team

## Implementation Notes

- Wrap Map instances in useRef() to persist across React render cycles without triggering re-renders on mutation
- Implement cleanup logic in useEffect return functions or explicit cleanup handlers to ensure Map.delete() is called when components unmount
- Use compound keys (e.g., `${zoneId}:${componentId}`) when managing state across multiple namespaces to prevent key collisions
- Consider exposing Map.size in development builds for debugging and memory leak detection during testing

## Continuation Context


Verify commands:
- grep -r 'Map<.*>' packages/core/components --include='*.tsx' --include='*.ts' | grep -E '(measureRefsRef|measuredItemHeights|rootVirtualizers)'
- grep -r '\.delete\(' packages/core/components --include='*.tsx' --include='*.ts' -A 2 -B 2
- grep -r 'useEffect.*return.*=>.*\.delete\(' packages/core/components --include='*.tsx' --include='*.ts'

Accept when:
- All Map instances used for component lifecycle tracking have corresponding .delete() calls in cleanup paths
- Map.get() and Map.set() operations are used consistently for component-keyed state access across virtualized components
- No memory leaks detected in profiling tests that mount/unmount virtualized components 1000+ times

## Enforcement

- Verified by: Code review checklist requiring cleanup logic verification for all Map usage
- Verified by: ESLint custom rule detecting Map.set() without corresponding .delete() in component scope
- Verified by: Memory profiling tests in CI pipeline measuring heap growth over component lifecycle iterations
- Violation handling: CI build fails if memory profiling tests detect reference accumulation exceeding threshold
- Violation handling: Code review blocks merge if Map cleanup logic is missing or incomplete
- Violation handling: Runtime warnings in development builds when Map.size() exceeds expected bounds
- Exception process: Document exception rationale in code comments explaining why cleanup is deferred or unnecessary
- Exception process: Obtain architecture review approval for exceptions with performance profiling evidence
- Exception process: Add exception to policy_exceptions section with specific conditions and approval requirements