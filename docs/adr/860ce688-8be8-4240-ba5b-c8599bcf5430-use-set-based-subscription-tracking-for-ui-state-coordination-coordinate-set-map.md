# Use Set-Based Subscription Tracking for UI State Coordination: Coordinate Set Map

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses @dnd-kit/react and @tanstack/react-virtual for drag-and-drop and virtualized list rendering, requiring coordination between UI interaction state and virtualization boundaries
- Components VirtualizedDropZone and DragDropContext manage ephemeral UI state including pinned indexes, measured item heights, and zone registrations that must be tracked across render cycles
- The store pattern (../../store) is imported by both components, suggesting centralized state management, but local subscription tracking uses Set data structures (nextPinnedIndexes.add, zoneStore.subscribe)
- Map-based caching (measuredItemHeights.get/set, rootVirtualizers.set/delete, measureRefsRef.current.get/set/delete) coordinates measurement data between virtualization and drag-drop systems

## Problem Statement

When coordinating drag-and-drop interactions with virtualized rendering, the system must track dynamic UI state (pinned indexes, zone registrations, item measurements) that changes frequently during user interaction. Traditional array-based or object-based tracking introduces O(n) lookup costs and duplicate entry risks, while Set and Map structures provide O(1) operations for membership testing and keyed access patterns required by virtualization and interaction boundaries.

## Decision

1. SHOULD: Coordinate Set/Map-based state updates with store subscription patterns (zoneStore.subscribe) to propagate changes across component boundaries

## Policy Block

- SHOULD Coordinate Set/Map-based state updates with store subscription patterns (zoneStore.subscribe) to propagate changes across component boundaries

In scope:
- UI interaction state in drag-and-drop contexts using @dnd-kit/react
- Virtualized list rendering state managed by @tanstack/react-virtual
- Component-local caches for measurements, registrations, and ephemeral UI coordination
- Store subscription callbacks that track zone depth indexes and preview states

Out of scope:
- Persistent application data stored in databases or external APIs
- Server-side state management or backend message queues
- Global application state that requires serialization or persistence
- Non-interactive data structures such as configuration objects or static lookup tables

Exceptions:
- EXC-001: Performance profiling demonstrates that array-based tracking with linear search is faster for collections guaranteed to remain under 10 items

## Rationale

- The evidence shows Set.add() used for nextPinnedIndexes and Map.get/set/delete used for measuredItemHeights and rootVirtualizers, indicating a deliberate choice for O(1) operations in performance-sensitive UI coordination paths
- Virtualized rendering with @tanstack/react-virtual requires frequent measurement lookups (measuredItemHeights.get(componentId)) during scroll and layout calculations, making Map structures essential for avoiding O(n) array scans
- The subscription pattern (zoneStore.subscribe) combined with Set/Map updates suggests a hybrid approach where local Set/Map state coordinates immediate UI updates while store subscriptions propagate changes to dependent components
- Explicit cleanup (measureRefsRef.current.delete, rootVirtualizers.delete) in the evidence demonstrates awareness of memory management requirements for long-lived virtualized lists with dynamic item sets

## Consequences

Positive:
- O(1) membership testing and keyed access eliminates performance bottlenecks in virtualized lists with hundreds or thousands of items
- Set data structures prevent duplicate entries automatically, reducing defensive programming and validation logic
- Map-based caching enables efficient coordination between drag-drop interaction state and virtualization measurement systems without full re-renders
- Explicit delete operations provide clear lifecycle management for ephemeral UI state tied to component mount/unmount cycles

Negative:
- Set and Map structures are not serializable by default, complicating state persistence, debugging with Redux DevTools, or server-side rendering scenarios
- Developers unfamiliar with Set/Map APIs may default to array methods, creating inconsistent patterns across the codebase
- Memory leaks can occur if delete operations are missed during component cleanup, especially in complex virtualized scenarios with nested zones
- Debugging Set/Map state requires console logging or custom tooling since browser DevTools provide limited inspection compared to plain objects

## Alternatives

- Use plain JavaScript objects with string keys for all UI state tracking (rejected)
  Rejected because: Object key lookup is O(1) but lacks Set semantics for unique membership, requires manual duplicate checking, and delete operator has performance implications in hot paths
  When valid: When state must be serialized for persistence or debugging, and collection sizes remain small (under 50 items)
- Use arrays with Array.includes() for membership testing and Array.find() for keyed lookups (rejected)
  Rejected because: O(n) lookup costs are unacceptable for virtualized lists with hundreds of items where measurements are accessed on every scroll event
  When valid: For small, static collections (under 10 items) where simplicity and serializability outweigh performance concerns
- Use Immer or immutable data structures for all UI state coordination (deferred)
  Rejected because: Not rejected but not adopted; immutable structures add overhead for ephemeral UI state that changes frequently and does not require time-travel debugging
  When valid: When UI state must integrate with Redux or other immutable state management systems, or when undo/redo functionality is required

## Risks

- Memory leaks from forgotten delete() calls in component cleanup, especially in nested virtualized zones with complex lifecycle dependencies
  Mitigation: Establish linting rules or custom hooks that enforce cleanup patterns; add memory profiling to CI for long-running UI tests
  Owner: Frontend engineering team
- Inconsistent adoption where some components use Set/Map while others use arrays, creating performance cliffs and maintenance confusion
  Mitigation: Document the pattern in component guidelines; provide code review checklist items for virtualized list implementations
  Owner: Tech lead and code reviewers
- Debugging difficulty when Set/Map state is not visible in standard React DevTools or Redux DevTools
  Mitigation: Add custom logging in store subscriptions (as seen in zoneStore.subscribe console.log); consider custom DevTools extensions or debug panels
  Owner: Developer experience team

## Implementation Notes

- Wrap Set/Map state in useRef when updates should not trigger re-renders (measureRefsRef.current pattern), but use useState when updates must trigger component updates
- Always pair Map.set() with corresponding Map.delete() in cleanup functions (useEffect return, component unmount) to prevent memory leaks
- Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers
- Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components
- For debugging, add temporary console.log statements in store subscriptions to inspect Set/Map state changes during development

## Continuation Context


Verify commands:
- grep -r 'new Set\|new Map\|\.add(\|\.set(\|\.delete(' packages/core/components --include='*.tsx' --include='*.ts'
- grep -r 'useRef.*Map\|useRef.*Set' packages/core/components --include='*.tsx' --include='*.ts'
- grep -r 'useEffect.*return.*delete\|componentWillUnmount.*delete' packages/core/components --include='*.tsx' --include='*.ts'

Accept when:
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones)
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries

## Enforcement

- Verified by: Code review checklist requiring Set/Map usage verification for virtualized list components
- Verified by: Automated grep-based checks in CI pipeline scanning for Set/Map patterns in components using @tanstack/react-virtual
- Verified by: Memory profiling in integration tests for drag-drop scenarios to detect leaks from missing delete() calls
- Violation handling: Code review feedback requesting refactor from array-based to Set/Map-based tracking for virtualized components
- Violation handling: CI warnings when Map.set() or Set.add() detected without corresponding delete() in same file
- Violation handling: Performance regression alerts if virtualized list scroll performance degrades below baseline thresholds
- Exception process: Submit exception request with performance profiling data demonstrating that alternative approach meets performance requirements
- Exception process: Tech lead approval required with documented rationale in component comments or ADR addendum
- Exception process: Exception review during quarterly architecture review to assess if pattern remains valid