# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Components Managing Virtualized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements virtualized drag-and-drop zones using @tanstack/react-virtual and @dnd-kit/react, requiring coordination between multiple UI components that manage scroll positions, item measurements, and drag state
- Component state must be synchronized across VirtualizedDropZone and DragDropContext components to maintain consistent preview indexes, zone depth tracking, and area depth tracking during drag operations
- Direct prop drilling or context-only patterns are insufficient for managing the dynamic, frequently-updating state of virtualized lists where items are measured on-demand and virtualizers are registered/unregistered dynamically
- The store module provides a centralized state management layer that components subscribe to, enabling reactive updates when drag operations affect multiple zones simultaneously

## Problem Statement

How should components in a virtualized drag-and-drop system coordinate state changes across zone boundaries when virtualizer handles, item measurements, and drag preview indexes must remain synchronized without introducing tight coupling between UI components?

## Decision

1. MUST: Components managing virtualized zones MUST subscribe to the zone store using the subscribe pattern to receive state updates for preview indexes, zone depth indexes, and area depth indexes

## Policy Block

- MUST Components managing virtualized zones MUST subscribe to the zone store using the subscribe pattern to receive state updates for preview indexes, zone depth indexes, and area depth indexes

In scope:
- VirtualizedDropZone components using @tanstack/react-virtual
- DragDropContext components coordinating @dnd-kit/react drag operations
- State synchronization between zone store subscribers
- Virtualizer handle registration and lifecycle management
- Item measurement caching and reference tracking

Out of scope:
- Non-virtualized drag-and-drop implementations
- Static list components without dynamic measurement requirements
- Single-zone drag operations without cross-zone coordination
- Server-side state synchronization or persistence

## Rationale

- The evidence shows explicit use of zoneStore.subscribe() with callbacks accessing previewIndex, zoneDepthIndex, and areaDepthIndex, demonstrating a pub-sub pattern for cross-component state coordination
- Map-based storage (rootVirtualizers.set/get/delete, measuredItemHeights.set/get, measureRefsRef.current.set/get/delete) provides O(1) lookups for virtualizer handles and measurements while maintaining clear ownership boundaries
- Set-based tracking (nextPinnedIndexes.add) prevents duplicate index processing during drag operations that may affect multiple items simultaneously
- The pattern decouples VirtualizedDropZone measurement logic from DragDropContext drag coordination, allowing each component to manage its domain-specific concerns while reacting to shared state changes

## Consequences

Positive:
- Components remain loosely coupled, subscribing only to the state slices they need without direct dependencies on other UI components
- Virtualizer handles and measurements can be registered/unregistered dynamically as zones mount/unmount without affecting other zones
- The subscription pattern enables multiple zones to react to the same drag operation simultaneously, supporting complex multi-zone drag scenarios
- Map and Set data structures provide efficient lookups and uniqueness guarantees for frequently-accessed state during scroll and drag operations

Negative:
- Subscription management adds complexity to component lifecycle, requiring careful cleanup to prevent memory leaks from orphaned subscriptions
- Debugging state flow becomes more difficult as updates propagate through subscriptions rather than explicit prop chains
- The centralized store creates a potential bottleneck if subscription callbacks perform expensive operations on every state change
- Implicit dependencies through subscriptions make it harder to understand component data requirements from reading component code alone

## Alternatives

- Use React Context with useContext hooks to share state between VirtualizedDropZone and DragDropContext (rejected)
  Rejected because: Context re-renders all consumers on any state change, causing performance issues in virtualized lists where measurements and scroll positions update frequently
  When valid: Appropriate for infrequent state updates or small component trees where re-render cost is negligible
- Pass virtualizer handles and measurement callbacks through props from parent to child components (rejected)
  Rejected because: Creates tight coupling between parent and child components and requires prop drilling through intermediate components that don't need the data
  When valid: Suitable for simple parent-child relationships with direct data flow and no intermediate components
- Use custom events (EventTarget/EventEmitter) to broadcast state changes between components (rejected)
  Rejected because: Lacks type safety and structured state management, making it difficult to track what events exist and what data they carry
  When valid: Acceptable for loosely-coupled systems where components have no shared state schema and events are infrequent

## Risks

- Memory leaks from subscriptions not properly cleaned up when components unmount, especially in dynamic lists where zones are frequently added/removed
  Mitigation: Enforce subscription cleanup in useEffect return functions, implement automated testing to detect orphaned subscriptions, add development-mode warnings for unmatched subscribe/unsubscribe calls
  Owner: engineering team
- Performance degradation if subscription callbacks trigger expensive re-renders or computations on every state update
  Mitigation: Use React.memo, useMemo, and useCallback to prevent unnecessary re-renders, implement subscription selectors to only trigger callbacks when relevant state slices change, profile subscription callback execution time
  Owner: engineering team
- Race conditions when multiple components update shared Map/Set structures concurrently during rapid drag operations
  Mitigation: Ensure all Map/Set mutations occur within React's state update cycle, use functional updates where order matters, add integration tests for concurrent drag scenarios
  Owner: engineering team

## Implementation Notes

- Store subscriptions in useEffect hooks with cleanup functions that call unsubscribe or remove handlers on component unmount
- Use Map.get() to check for existing entries before Map.set() operations to avoid overwriting valid data, and always call Map.delete() in cleanup
- Wrap subscription callbacks with useCallback to prevent subscription churn from callback reference changes on every render
- Consider implementing a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change, reducing unnecessary updates
- Add TypeScript types for subscription callback signatures to ensure type safety across the store boundary

## Continuation Context


Verify commands:
- grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'
- grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'
- grep -r 'useEffect.*return.*\(delete\|unsubscribe\)' packages/core/components/ --include='*.tsx'

Accept when:
- All components using virtualized zones contain zoneStore.subscribe() calls within useEffect hooks
- All Map.set() operations for virtualizer handles and measurements have corresponding Map.delete() calls in cleanup functions
- Subscription callbacks are wrapped in useCallback or defined outside render to prevent unnecessary subscription churn

## Enforcement

- Verified by: Code review checklist requiring subscription cleanup verification for all new virtualized components
- Verified by: ESLint custom rule detecting subscribe() calls without corresponding cleanup in useEffect return
- Verified by: Integration tests verifying no memory leaks after mounting/unmounting virtualized zones 100 times
- Violation handling: CI pipeline fails if grep verification commands detect subscribe() without cleanup patterns
- Violation handling: Memory leak detection tests fail the build if subscription count increases after component unmount cycles
- Violation handling: Code review blocks merge if Map/Set operations lack corresponding cleanup in component lifecycle
- Exception process: Document the specific technical constraint preventing standard subscription cleanup (e.g., third-party library limitation)
- Exception process: Propose alternative cleanup mechanism or compensating control to prevent memory leaks
- Exception process: Obtain approval from tech lead with explicit acceptance of the risk and mitigation plan