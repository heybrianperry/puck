# Standardize React Component Export as Public API Contracts in Data Access Layer: Components Coordinating Data

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React components as primary public API contracts, exporting components like VirtualizedDropZone, DragDropContext, AutoFrame, ViewportControls, and Layout from packages/core/components
- Components coordinate data access through multiple patterns: Map-based caching (measuredItemHeights.get/set, rootVirtualizers.set/delete), array operations (Array.from().find()), and subscription-based state management (zoneStore.subscribe)
- The pattern emerges from a component library architecture where UI components encapsulate both presentation logic and data access coordination, using React hooks (useCallback, useMemo, useEffect, useState) as the primary abstraction layer
- Dependencies on @dnd-kit/react, @tanstack/react-virtual, and react-dom indicate specialized UI interaction patterns that require coordinated state management and caching strategies
- The architecture separates concerns through context providers (autoFrameContext, useFrame) and store abstractions (../../store imports) while maintaining component-level API boundaries

## Problem Statement

Components in the data access layer need a consistent pattern for exposing public APIs while coordinating multiple data access strategies (Map-based caching, array queries, subscriptions) without leaking implementation details or creating tight coupling between UI interactions and underlying data structures.

## Decision

1. MUST: Components coordinating data access MUST use Map-based caching for keyed lookups (e.g., measuredItemHeights.get/set, rootVirtualizers.set/delete) when managing component-scoped state

## Policy Block

- MUST Components coordinating data access MUST use Map-based caching for keyed lookups (e.g., measuredItemHeights.get/set, rootVirtualizers.set/delete) when managing component-scoped state

In scope:
- All React components in packages/core/components that export public APIs
- Components using Map-based caching for component-scoped state (measuredItemHeights, rootVirtualizers, measureRefsRef)
- Components coordinating data access through hooks (useCallback, useMemo, useEffect, useState)
- Components integrating with external libraries (@dnd-kit/react, @tanstack/react-virtual) for data-driven UI interactions

Out of scope:
- Internal utility functions not exported as public APIs
- Server-side data access patterns or API routes
- Non-React data access implementations
- Build-time or compile-time data transformations

Exceptions:
- EXC-001: Legacy components with established external consumers may defer TypeScript type exports until next major version
- EXC-002: Performance-critical paths may expose internal data structures if profiling demonstrates measurable benefit (>10% improvement)

## Rationale

- Evidence shows 5 files consistently exporting React components as public API contracts (VirtualizedDropZone, DragDropContext, AutoFrame, ViewportControls, Layout) with 85.76% pattern confidence
- Map-based caching (measuredItemHeights.get/set, rootVirtualizers.set/delete) provides O(1) lookup performance for component-scoped state while maintaining referential stability across renders
- Array.find() operations appear in data access paths (path.find, defaultZoomOptions.find, plugins?.find) requiring memoization to prevent performance degradation
- The pattern coordinates multiple concerns (UI interactions, caching, subscriptions) through React's compositional model, enabling testability and reusability while maintaining clear API boundaries

## Consequences

Positive:
- Clear API boundaries through typed component exports enable safe consumption by external packages and applications
- Map-based caching provides predictable O(1) performance for component state lookups without introducing complex state management libraries
- Hook-based encapsulation (useCallback, useMemo) prevents unnecessary recomputation and enables fine-grained optimization
- Separation of data access from presentation through context providers and custom hooks improves testability and maintainability

Negative:
- Multiple data access patterns (Map caching, array queries, subscriptions) increase cognitive load for developers unfamiliar with the component internals
- React hook dependencies and memoization require careful management to avoid stale closures or infinite render loops
- Map-based caching adds memory overhead for long-lived components with large datasets
- Component-level API boundaries may obscure underlying data access inefficiencies until performance profiling is conducted

## Alternatives

- Use a centralized state management library (Redux, MobX) for all data access coordination instead of component-scoped Maps and hooks (rejected)
  Rejected because: Evidence shows the codebase already uses component-scoped state and store abstractions (../../store imports); introducing a global state library would require significant refactoring and may not align with the component library's encapsulation goals
  When valid: Valid for applications with complex cross-component data dependencies or time-travel debugging requirements
- Expose raw data access primitives (Map, Set, subscription handles) directly in component props for maximum flexibility (rejected)
  Rejected because: Violates encapsulation principles and creates tight coupling between consumers and internal implementation details, making refactoring and optimization difficult
  When valid: Valid only for internal utility components not exposed as public APIs
- Implement a custom data access layer abstraction separate from React components (deferred)
  Rejected because: Would require significant architectural changes; current pattern is functional with 85.76% confidence across 5 files
  When valid: Valid if the codebase scales beyond component library scope or requires non-React data access patterns

## Risks

- Map-based caching without size limits could cause memory leaks in long-running applications with dynamic component creation
  Mitigation: Implement cache eviction policies or WeakMap usage for component-scoped caches; add monitoring for cache size growth
  Owner: engineering team
- Array.find() operations on large datasets could degrade performance if not properly memoized or if datasets grow unexpectedly
  Mitigation: Add performance budgets and profiling for array operations; consider indexed data structures (Map) for frequently queried arrays
  Owner: engineering team
- Subscription-based patterns (zoneStore.subscribe) without proper cleanup could cause memory leaks or stale state updates
  Mitigation: Enforce useEffect cleanup functions for all subscriptions; add linting rules to detect missing cleanup
  Owner: engineering team

## Implementation Notes

- Use TypeScript's export type syntax for component props interfaces to ensure clear API contracts (e.g., export type AutoFrameProps = {...})
- Wrap Map.get/set operations in useCallback or useMemo hooks to maintain referential stability and prevent unnecessary re-renders
- For array.find() operations, consider converting frequently queried arrays to Map structures for O(1) lookup performance
- Always return cleanup functions from useEffect hooks when subscribing to stores or setting up event listeners (e.g., return () => rootVirtualizers.delete(zoneCompound))
- Document cache eviction strategies in component documentation when using Map-based caching for unbounded datasets

## Continuation Context


Verify commands:
- grep -r "export.*Props" packages/core/components --include="*.tsx" --include="*.ts" | wc -l
- grep -r "\.get(\|\.set(\|\.delete(" packages/core/components --include="*.tsx" | grep -E "(measuredItemHeights|rootVirtualizers|measureRefsRef)" | wc -l
- grep -r "useCallback\|useMemo\|useEffect" packages/core/components --include="*.tsx" | wc -l

Accept when:
- All public API components export TypeScript type definitions for their props interfaces
- Map-based caching operations (get/set/delete) are present in components coordinating data access
- React hooks (useCallback, useMemo, useEffect) are used to encapsulate data access logic and prevent unnecessary recomputation

## Enforcement

- Verified by: TypeScript compilation enforces exported type definitions for public API components
- Verified by: Code review checklist includes verification of Map-based caching patterns and hook usage
- Verified by: ESLint rules enforce useEffect cleanup functions for subscriptions and side effects
- Violation handling: TypeScript compilation failures block PR merges for missing type exports
- Violation handling: Code review feedback requires refactoring of direct array.find() usage in render paths to memoized hooks
- Violation handling: Performance regression tests flag components with uncached data access patterns exceeding budget thresholds
- Exception process: Submit exception request with performance benchmarks or migration timeline to architecture review board
- Exception process: Document exception rationale in component documentation and ADR exception log
- Exception process: Set expiration date for temporary exceptions with required follow-up tasks