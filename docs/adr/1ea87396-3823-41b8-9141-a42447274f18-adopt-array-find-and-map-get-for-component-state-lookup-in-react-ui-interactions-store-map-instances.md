# Adopt Array.find() and Map.get() for Component State Lookup in React UI Interactions: Store Map Instances

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React components in packages/core require efficient lookup of component state, measurements, and virtualizer handles across drag-drop zones, viewport controls, and frame management
- The codebase uses @dnd-kit/react, @tanstack/react-virtual, and react-dom for complex UI interactions requiring indexed access to component references and cached measurements
- Map-based caches (measuredItemHeights, measureRefsRef, rootVirtualizers) and Array.find() operations appear in 5 files with 85.76% pattern consistency
- Components coordinate state through useCallback, useMemo, and useEffect hooks that depend on predictable lookup semantics for component IDs and zone identifiers
- The pattern emerged to support virtualized rendering, drag-drop coordination, and frame stylesheet mirroring where direct object property access is insufficient

## Problem Statement

React components managing complex UI interactions (virtualized lists, drag-drop zones, iframe style mirroring) require consistent, type-safe mechanisms to retrieve component state, measurements, and references by identifier. Without standardized lookup patterns, components risk inconsistent cache access, failed reference resolution, and unpredictable behavior during state updates and effect reconciliation.

## Decision

1. SHOULD: Store Map instances in useRef containers (e.g., measureRefsRef.current) when lookup state must persist across renders without triggering re-renders

## Policy Block

- SHOULD Store Map instances in useRef containers (e.g., measureRefsRef.current) when lookup state must persist across renders without triggering re-renders

In scope:
- React components in packages/core managing drag-drop interactions (@dnd-kit/react)
- Virtualized list components using @tanstack/react-virtual with dynamic item measurements
- AutoFrame components mirroring stylesheets from parent documents into iframes
- Viewport controls and layout components coordinating zoom options and plugin state
- Any component using useRef-stored Map instances for component ID-indexed caches

Out of scope:
- Server-side rendering or static generation contexts where DOM collections are unavailable
- Simple prop-based lookups where component identity is statically known at compile time
- Third-party library internals that provide their own lookup abstractions
- Non-React UI frameworks or vanilla JavaScript modules

## Rationale

- Map.get() provides O(1) lookup performance for component measurements and references indexed by string identifiers (componentId, zoneCompound), critical for virtualized rendering performance
- Array.find() offers type-safe, predicate-based search for collections where identity depends on property comparison (path matching in drag-drop, href matching in stylesheets, value matching in zoom options)
- The pattern appears consistently across 5 files with 85.76% confidence, indicating established architectural convention in the packages/core codebase
- Explicit Map.delete() cleanup in useEffect return functions prevents memory leaks in components with dynamic lifecycles (VirtualizedDropZone, DragDropContext)

## Consequences

Positive:
- Consistent lookup semantics across drag-drop, virtualization, and frame management components reduce cognitive load and improve maintainability
- Map-based caches provide predictable O(1) performance for component reference and measurement retrieval in high-frequency interaction handlers
- Array.find() with explicit predicates improves code readability and type safety compared to manual iteration or index-based access
- Explicit cleanup with Map.delete() in useEffect return functions prevents memory leaks in long-lived component trees with dynamic children

Negative:
- Map.get() returns undefined for missing keys, requiring null checks or optional chaining in consuming code
- Array.find() has O(n) complexity and may degrade performance on large collections without memoization or indexing
- Storing Maps in useRef requires careful synchronization with React's render cycle to avoid stale reads during concurrent rendering
- Pattern requires developers to understand Map API and predicate functions, increasing onboarding complexity for junior engineers

## Alternatives

- Use plain JavaScript objects with bracket notation for component ID-indexed caches (rejected)
  Rejected because: Objects lack explicit .delete() semantics for cleanup, risk prototype pollution with dynamic keys, and provide no type safety for key-value relationships
  When valid: Acceptable for static, compile-time-known property names in simple prop-based lookups
- Use Array.filter().at(0) instead of Array.find() for predicate-based search (rejected)
  Rejected because: Array.filter() allocates intermediate arrays and iterates entire collection even after match is found, degrading performance in large collections
  When valid: Acceptable when multiple matches are needed or when filter result is reused for other operations
- Store lookup state in React useState instead of useRef-wrapped Maps (rejected)
  Rejected because: useState triggers re-renders on every Map.set() operation, causing unnecessary reconciliation in high-frequency interaction handlers like drag-drop and scroll
  When valid: Valid when lookup state changes must trigger component re-renders and updates are infrequent

## Risks

- Array.find() on large collections (e.g., document.styleSheets, plugin arrays) may degrade performance if called in high-frequency event handlers without memoization
  Mitigation: Wrap Array.find() results in useMemo with appropriate dependencies, or maintain reverse lookup Maps for O(1) access
  Owner: engineering team
- Map instances stored in useRef may become stale or inconsistent during React concurrent rendering if updates are not properly synchronized
  Mitigation: Use useEffect to synchronize Map updates with component lifecycle, and avoid reading Map state during render phase
  Owner: engineering team
- Missing Map.delete() cleanup in component unmount handlers causes memory leaks in long-lived applications with dynamic component trees
  Mitigation: Enforce cleanup in useEffect return functions via code review and linting rules, add memory profiling to CI for leak detection
  Owner: engineering team

## Implementation Notes

- Store Map instances in useRef when lookup state must persist across renders without triggering re-renders: const mapRef = useRef(new Map())
- Always return cleanup functions from useEffect that call Map.delete() for component-scoped cache entries to prevent memory leaks
- Convert DOM collections to arrays before using .find(): Array.from(document.styleSheets).find(predicate)
- Wrap Array.find() results in useMemo when searching large collections or when the result is used in render or effect dependencies
- Use optional chaining or null checks when accessing Map.get() results: const value = map.get(key)?.property

## Continuation Context


Verify commands:
- grep -r 'Map\.get(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'Array\.find(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'Map\.delete(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'useRef.*new Map' packages/core/components --include='*.tsx' --include='*.ts'

Accept when:
- Map.get() and Map.delete() calls are present in at least 3 component files in packages/core/components
- Array.find() with predicate functions is used for collection searches in at least 3 component files
- useEffect return functions include Map.delete() cleanup for component-scoped cache entries in virtualized or drag-drop components

## Enforcement

- Verified by: Code review checklist requiring Map.get()/Array.find() pattern verification for new UI interaction components
- Verified by: ESLint custom rules detecting direct object property access in component ID lookup contexts
- Verified by: Automated grep-based verification in CI pipeline checking for Map.delete() in useEffect cleanup functions
- Violation handling: CI pipeline fails if new components use object bracket notation for component ID-indexed caches without justification
- Violation handling: Code review blocks merges lacking Map.delete() cleanup in components with dynamic Map-based state
- Violation handling: Memory profiling tests flag components with growing Map instances that lack proper cleanup
- Exception process: Document exception rationale in code comments explaining why alternative lookup pattern is required
- Exception process: Obtain approval from frontend architecture team for exceptions in performance-critical paths
- Exception process: Add exception to .eslintrc with inline comment linking to ADR and justification