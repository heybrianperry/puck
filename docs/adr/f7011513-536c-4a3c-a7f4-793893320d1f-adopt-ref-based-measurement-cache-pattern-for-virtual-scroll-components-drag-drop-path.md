# Adopt Ref-Based Measurement Cache Pattern for Virtual Scroll Components: Drag Drop Path

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Virtual scroll components in the packages/core module require dynamic measurement of item heights to calculate scroll positions and viewport rendering boundaries
- The @tanstack/react-virtual library integration necessitates coordination between React component lifecycle (useEffect, useCallback, useMemo) and imperative measurement APIs
- The DragDropContext and VirtualizedDropZone components manage stateful collections of measurements and virtualizer handles that must persist across re-renders but remain accessible for imperative cleanup
- The @dnd-kit/react and @dnd-kit/dom integration requires synchronization between drag-drop event paths and virtualized zone registries stored in mutable ref containers

## Problem Statement

Virtual scroll and drag-drop components require imperative access to measurement data and virtualizer handles that must persist across React re-renders without triggering unnecessary updates, while supporting dynamic registration, lookup, mutation, and cleanup operations coordinated with component lifecycle events.

## Decision

1. SHOULD: Drag-drop path resolution SHOULD use Array.find with destructured path identifiers to locate source zones within virtualized hierarchies

## Policy Block

- SHOULD Drag-drop path resolution SHOULD use Array.find with destructured path identifiers to locate source zones within virtualized hierarchies

## Rationale

- The evidence shows consistent use of Map.get(), Map.set(), and Map.delete() operations on ref-stored collections (measuredItemHeights, measureRefsRef.current, rootVirtualizers) across both VirtualizedDropZone.tsx and DragDropContext/index.tsx
- React hooks (useCallback, useMemo, useEffect) coordinate imperative cache operations with declarative component lifecycle, preventing measurement data from entering React's state reconciliation path
- The @tanstack/react-virtual and @dnd-kit integration requires mutable handles that survive re-renders but remain accessible for synchronous lookup during scroll calculations and drag event routing
- The pattern appears in 2 files with 87.10% confidence, indicating a deliberate architectural choice for managing stateful imperative APIs within React's declarative model

## Consequences

Positive:
- Measurement cache operations execute without triggering React re-renders, improving performance for high-frequency scroll and drag events
- Map-based registries provide O(1) lookup and deletion performance for component identifier-keyed data
- Ref-based storage maintains referential stability across renders while supporting imperative cleanup in useEffect
- Clear separation between declarative React state and imperative measurement APIs reduces coupling to virtualization library internals

Negative:
- Imperative cache mutations bypass React's state management, requiring manual synchronization with component lifecycle
- Ref-based patterns increase cognitive load for developers unfamiliar with mixing declarative and imperative React patterns
- Debugging measurement cache state requires direct inspection of ref.current values, which are not visible in React DevTools
- Cleanup logic must be manually coordinated across multiple useEffect hooks, increasing risk of memory leaks if delete() calls are missed

## Alternatives

- Store measurement data in React useState and trigger re-renders on every measurement update (rejected)
  Rejected because: High-frequency measurement updates during scroll would trigger excessive re-renders, degrading performance and causing layout thrashing
  When valid: Only valid for low-frequency measurement scenarios where re-render cost is negligible
- Use external state management library (Redux, Zustand) for measurement cache (rejected)
  Rejected because: Adds dependency overhead and still requires imperative access patterns for synchronous measurement lookup during scroll calculations
  When valid: Valid if measurements need to be shared across distant component trees or persisted beyond component lifecycle
- Delegate all measurement caching to @tanstack/react-virtual internal state (rejected)
  Rejected because: Library does not expose sufficient control over measurement lifecycle for drag-drop coordination and custom cleanup requirements
  When valid: Valid for simple virtualization scenarios without drag-drop integration or custom measurement invalidation logic

## Risks

- Memory leaks if measurement cache delete() operations are not called during component unmount or identifier changes
  Mitigation: Enforce useEffect cleanup functions that call .delete() for all registered componentId and zoneCompound keys; add ESLint rules to verify cleanup patterns
  Owner: engineering team
- Race conditions between measurement updates and virtualizer scroll calculations if cache mutations occur outside React lifecycle
  Mitigation: Restrict cache mutations to useEffect and useCallback scopes; document synchronization requirements in component interfaces
  Owner: engineering team
- Stale measurement data if cache is not invalidated when component dimensions change due to external factors (window resize, CSS changes)
  Mitigation: Implement ResizeObserver integration to trigger cache invalidation; add verification commands to detect stale measurements
  Owner: engineering team

## Implementation Notes

- Initialize measurement caches as useRef(new Map()) at component top level to ensure single Map instance per component lifecycle
- Wrap all cache access operations (get, set, delete) in useCallback hooks to maintain referential stability for child component props
- Structure useEffect cleanup functions to iterate over all registered keys and call .delete() before component unmount
- Use TypeScript generics to type Map keys and values (e.g., Map<string, number> for measuredItemHeights) to prevent runtime type errors during cache operations

## Continuation Context


Verify commands:
- grep -r 'useRef.*Map' packages/core/components --include='*.tsx' | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)'
- grep -r '\.delete\(' packages/core/components --include='*.tsx' -A 2 -B 2 | grep -E '(useEffect|cleanup)'
- grep -r 'Map\.(get|set|delete)' packages/core/components --include='*.tsx' | wc -l

Accept when:
- All virtual scroll components use useRef(new Map()) for measurement caches and virtualizer registries
- Every Map.set() operation has a corresponding Map.delete() in a useEffect cleanup function or unmount handler
- Grep commands identify at least 2 files with Map-based ref patterns matching the evidence structure

## Enforcement

- Verified by: Code review checklist verifying useEffect cleanup functions call .delete() for all registered cache keys
- Verified by: ESLint custom rule detecting useRef(new Map()) without corresponding cleanup logic
- Verified by: Integration tests asserting measurement cache size returns to zero after component unmount
- Violation handling: CI pipeline fails if grep verification commands do not match expected pattern counts
- Violation handling: Code review blocks merge if Map-based caches lack cleanup functions
- Violation handling: Runtime warnings logged in development mode when cache size exceeds threshold after component unmount
- Exception process: Document exception rationale in component JSDoc explaining why cleanup is deferred or omitted
- Exception process: Obtain approval from tech lead for components with non-standard cache lifecycle requirements
- Exception process: Add inline comments with EXCEPTION: prefix explaining deviation from standard pattern