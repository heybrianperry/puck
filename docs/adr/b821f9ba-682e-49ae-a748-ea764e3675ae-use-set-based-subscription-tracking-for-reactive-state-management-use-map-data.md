# Use Set-Based Subscription Tracking for Reactive State Management: Use Map Data

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React with @dnd-kit for drag-and-drop interactions requiring real-time state synchronization across virtualized components
- VirtualizedDropZone and DragDropContext components manage dynamic collections of items with measured heights and pinned indexes that change during user interactions
- State updates from drag operations need to propagate to multiple subscribers without triggering unnecessary re-renders in unaffected components
- The @tanstack/react-virtual library requires coordination with custom store implementations to maintain scroll position and item measurements during virtualization

## Problem Statement

Components managing virtualized drag-and-drop interactions require a mechanism to track which items are pinned, which virtualizers are active, and which measurements are cached, while ensuring that state changes propagate efficiently to subscribers without coupling the state management logic to specific UI framework patterns or causing performance degradation through excessive re-renders.

## Decision

1. MUST: Use Map data structures (e.g., measuredItemHeights.get/set/delete, rootVirtualizers.set/delete) for key-value associations where components or items are indexed by compound identifiers

## Policy Block

- MUST Use Map data structures (e.g., measuredItemHeights.get/set/delete, rootVirtualizers.set/delete) for key-value associations where components or items are indexed by compound identifiers

## Rationale

- Set and Map data structures provide O(1) membership testing and insertion, which is critical for performance when tracking dynamic collections during drag operations
- The subscription pattern (zoneStore.subscribe) decouples state producers from consumers, enabling multiple components to react to state changes without direct coupling
- Evidence shows explicit cleanup operations (delete) in both files, indicating awareness of memory management requirements in long-lived applications
- The pattern appears in both VirtualizedDropZone and DragDropContext components with 87.10% confidence across 2 files, suggesting intentional architectural consistency

## Consequences

Positive:
- Efficient O(1) lookup and insertion performance for tracking pinned indexes and measured heights during virtualization
- Decoupled state management allows independent evolution of store implementation and UI components
- Explicit cleanup operations prevent memory leaks in long-running applications with dynamic component lifecycles
- Subscription pattern enables multiple consumers to react to state changes without prop drilling or context pollution

Negative:
- Set and Map structures are not directly serializable to JSON, complicating state persistence or debugging workflows
- Subscription callbacks require manual cleanup to avoid memory leaks, increasing cognitive load for developers
- The pattern introduces indirection between state updates and UI rendering, making data flow harder to trace in debugging tools
- Compound key strategies require consistent key generation logic across components to avoid mismatches

## Alternatives

- Use plain JavaScript arrays with indexOf/includes for membership testing and push for additions (rejected)
  Rejected because: Array operations have O(n) complexity for membership testing, causing performance degradation with large collections during drag operations
  When valid: Valid only for small collections (< 10 items) where performance is not critical
- Use React Context with useState for state management instead of custom store with subscriptions (rejected)
  Rejected because: Context updates trigger re-renders in all consumers regardless of whether their specific data changed, causing unnecessary rendering overhead in virtualized lists
  When valid: Valid for small component trees with infrequent updates where re-render cost is negligible
- Use external state management library (Redux, Zustand, Jotai) with built-in subscription mechanisms (deferred)
  Rejected because: Not rejected; evidence shows custom store implementation but does not indicate whether external libraries were considered
  When valid: Valid when standardization across multiple state domains justifies the dependency cost

## Risks

- Memory leaks if subscription cleanup is missed during component unmounting or item removal
  Mitigation: Enforce useEffect cleanup functions that call Map.delete() and unsubscribe callbacks; add ESLint rules to detect missing cleanup
  Owner: engineering team
- Inconsistent compound key generation across components leading to orphaned Map entries or failed lookups
  Mitigation: Centralize compound key generation in shared utility functions; document key format in component interfaces
  Owner: engineering team
- Debugging difficulty due to non-serializable Set/Map structures in React DevTools and state snapshots
  Mitigation: Implement custom serialization helpers for logging; use Map/Set-aware debugging utilities or browser extensions
  Owner: engineering team

## Implementation Notes

- Store Map and Set references in useRef hooks to maintain stable references across React re-renders without triggering effects
- Always pair Map.set() or Set.add() operations with corresponding delete() calls in useEffect cleanup functions
- When implementing store.subscribe(), return an unsubscribe function and call it in the cleanup phase of useEffect
- Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures

## Continuation Context


Verify commands:
- grep -r '\.add(' packages/core/components/ | grep -v '\.delete(' | wc -l
- grep -r 'subscribe(' packages/core/components/ | grep -E 'useEffect|cleanup'
- grep -r 'Map\|Set' packages/core/components/ --include='*.tsx' --include='*.ts'

Accept when:
- All Set.add() and Map.set() operations have corresponding delete() calls in component cleanup or removal paths
- All store.subscribe() calls return unsubscribe functions that are invoked in useEffect cleanup
- Map and Set usage is documented with key format specifications in component interfaces or type definitions

## Enforcement

- Verified by: Code review checklist requiring verification of cleanup operations for all Map/Set mutations
- Verified by: ESLint custom rules detecting Set.add/Map.set without corresponding delete in same component
- Verified by: Unit tests asserting that subscription cleanup functions are called on component unmount
- Violation handling: CI pipeline fails if ESLint rules detect missing cleanup operations
- Violation handling: Code review blocks merge if Map/Set usage lacks documented key format or cleanup logic
- Violation handling: Runtime warnings in development mode when subscriptions are not cleaned up within expected lifecycle
- Exception process: Document exception rationale in code comments explaining why cleanup is not required (e.g., singleton stores with application lifetime)
- Exception process: Obtain approval from tech lead for exceptions involving intentional memory retention patterns
- Exception process: Add exception to ESLint ignore list with inline comment linking to architectural decision documentation