# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Components Use Set

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React components in packages/core/components manage virtualized rendering with dynamic component lifecycles requiring explicit cleanup of measurement references and virtualizer handles
- The codebase uses @tanstack/react-virtual and @dnd-kit/react for drag-drop and virtualization, necessitating manual registration and deregistration of component-specific state
- VirtualizedDropZone.tsx and DragDropContext/index.tsx both implement Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) to track component instances by ID
- Component unmounting and ID changes trigger explicit delete operations on these Map structures to prevent memory leaks and stale references in long-lived UI sessions

## Problem Statement

In React applications with virtualized lists and drag-drop interactions, component instances require registration of measurement callbacks, virtualizer handles, and cached dimensions. Without explicit lifecycle management, these registrations accumulate in memory as components mount and unmount, causing memory leaks and stale reference bugs. The system needs a consistent pattern for registering component-specific resources and cleaning them up when components are removed or their identifiers change.

## Decision

1. MAY: Components MAY use Set data structures for simpler boolean membership tracking (e.g., nextPinnedIndexes.add(currentIndex)) when resource cleanup is not required

## Policy Block

- MAY Components MAY use Set data structures for simpler boolean membership tracking (e.g., nextPinnedIndexes.add(currentIndex)) when resource cleanup is not required

## Rationale

- The evidence shows explicit Map.delete() calls in VirtualizedDropZone (measureRefsRef.current.delete(componentId)) and DragDropContext (rootVirtualizers.delete(zoneCompound)), demonstrating intentional cleanup patterns
- Map structures provide O(1) lookup and deletion by component ID, essential for performance in virtualized lists with hundreds of components mounting and unmounting during scrolling
- The pattern appears in both measurement caching (measuredItemHeights) and service registration (rootVirtualizers), indicating a general-purpose lifecycle management approach
- Using Map.get/set/delete provides type-safe, explicit lifecycle boundaries compared to object property access, reducing bugs from undefined references

## Consequences

Positive:
- Prevents memory leaks by ensuring component-specific resources are cleaned up when components unmount or IDs change
- Provides O(1) performance for registration, lookup, and cleanup operations even with large numbers of tracked components
- Enables explicit lifecycle management visible in code review, making resource ownership and cleanup responsibilities clear
- Supports dynamic component IDs and re-keying scenarios common in virtualized lists and drag-drop interfaces

Negative:
- Requires manual cleanup code in every component using the pattern, increasing boilerplate and risk of forgetting cleanup
- Map registries must be carefully scoped (refs vs module-level) to avoid unintended sharing or loss of state across renders
- Debugging lifecycle issues requires inspecting Map contents, which are not visible in React DevTools component state
- Pattern creates implicit coupling between component IDs and registry keys, requiring consistent ID generation and propagation

## Alternatives

- Use WeakMap with component instance objects as keys for automatic garbage collection (rejected)
  Rejected because: WeakMap requires object keys, but the evidence shows string-based component IDs (componentId, zoneCompound) are used for keying, making WeakMap incompatible with the existing ID-based architecture
  When valid: Valid when component instances themselves (not IDs) are the natural keys and automatic GC is preferred over explicit cleanup
- Store resources in React state or context to leverage automatic cleanup on unmount (rejected)
  Rejected because: Measurement callbacks and virtualizer handles are mutable imperative resources that should not trigger re-renders; storing in state would cause performance issues in virtualized lists
  When valid: Valid for declarative data that should trigger re-renders when changed, not for imperative handles or callbacks
- Use a custom hook (e.g., useComponentRegistry) to encapsulate Map lifecycle management (deferred)
  When valid: Valid as a refactoring to reduce boilerplate and centralize cleanup logic; could be adopted after pattern stabilizes across more components

## Risks

- Developers may forget to call Map.delete() in cleanup functions, causing memory leaks that accumulate over long user sessions
  Mitigation: Implement ESLint rules to detect Map.set() calls without corresponding delete() in useEffect cleanup; add memory profiling tests for virtualized components
  Owner: Frontend Engineering Team
- Component ID changes without cleanup (e.g., during re-keying) leave orphaned entries in Map registries
  Mitigation: Ensure useEffect dependencies include componentId so cleanup runs on ID changes; document ID stability requirements in component APIs
  Owner: Frontend Engineering Team
- Shared Map registries across component instances create coupling and potential race conditions during concurrent cleanup
  Mitigation: Document Map registry ownership and access patterns; use unique compound keys (e.g., zoneCompound) to prevent collisions; consider per-instance registries for isolated components
  Owner: Architecture Team

## Implementation Notes

- Store Map registries in useRef().current or module-level variables to persist across renders without causing re-renders
- Always include componentId in useEffect dependency arrays to ensure cleanup runs when IDs change, not just on unmount
- Use compound keys (e.g., `${zoneId}:${componentId}`) when multiple components share a registry to prevent key collisions
- Verify Map.get() returns a value before using it to handle race conditions where cleanup may have already occurred

## Continuation Context


Verify commands:
- grep -r 'Map.*\.set(' packages/core/components/ | xargs -I {} sh -c 'echo {} && grep -A 20 "{}" | grep -E "(useEffect|delete)"'
- grep -r '\.delete(' packages/core/components/ --include='*.tsx' --include='*.ts' -B 5 | grep -E '(componentId|zoneCompound)'
- npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cleanup|unmount|lifecycle'

Accept when:
- All Map.set() calls in component lifecycle code have corresponding Map.delete() calls in useEffect cleanup functions or component unmount handlers
- Grep verification shows componentId or equivalent keys are used consistently in .get(), .set(), and .delete() operations within the same component
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components

## Enforcement

- Verified by: Code review checklist requiring verification of Map cleanup in components using virtualization or drag-drop
- Verified by: ESLint custom rule detecting Map.set() without corresponding delete() in useEffect cleanup
- Verified by: Automated memory profiling tests in CI for components with Map-based registries
- Violation handling: CI build fails if ESLint detects Map.set() without cleanup in new or modified components
- Violation handling: Code review blocks merge if Map lifecycle management is incomplete or inconsistent
- Violation handling: Memory leak detection in integration tests triggers alerts and requires investigation before release
- Exception process: Document justification for Map entries that intentionally persist beyond component lifecycle (e.g., global caches)
- Exception process: Add eslint-disable comments with explanation for cases where cleanup is handled by parent components or external systems
- Exception process: Architecture team review required for new patterns that deviate from Map-based lifecycle management