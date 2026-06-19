# Adopt Array.find() and Map.get() for Collection Queries in React Components: Use Map Get

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React components in the packages/core module require frequent lookups of items from arrays and Map collections based on identity or predicate matching
- The codebase uses @dnd-kit/react, @tanstack/react-virtual, and custom state management requiring efficient item retrieval from collections stored in refs and state
- Components like VirtualizedDropZone, DragDropContext, AutoFrame, ViewportControls, and Layout implement UI interactions that depend on finding specific elements from collections
- The pattern emerged across 5 files with 85.76% confidence, indicating consistent adoption of Array.find() and Map.get() for data access
- The architecture separates cache layers (measuredItemHeights Map) from UI interaction logic, requiring predictable query patterns for component state

## Problem Statement

React components need a consistent, type-safe method to query collections (arrays and Maps) for specific items during render cycles, event handlers, and effect callbacks without introducing performance overhead or mutation side effects.

## Decision

1. MUST: Use Map.get() for keyed lookups in Map collections rather than iterating over entries

## Policy Block

- MUST Use Map.get() for keyed lookups in Map collections rather than iterating over entries

In scope:
- React functional components using hooks (useCallback, useMemo, useEffect, useState)
- Cache layers implemented with Map collections (measuredItemHeights, rootVirtualizers, measureRefsRef)
- Array queries over configuration options, plugin lists, and DOM collections
- Event handlers and effect callbacks requiring item lookups

Out of scope:
- Server-side data fetching or database queries
- Large-scale data processing requiring indexed search structures
- Performance-critical loops where manual iteration is measurably faster
- External library APIs that require specific iteration patterns

Exceptions:
- EXC-001: Performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with large collections (>1000 items)

## Rationale

- Evidence shows consistent use of Array.find() across 5 files for querying arrays of plugins, zoom options, stylesheets, and path elements, indicating established pattern adoption
- Map.get() and Map.set() appear in cache layer implementations (measuredItemHeights, rootVirtualizers, measureRefsRef) demonstrating preference for Map-based keyed access over object property access
- The pattern integrates with React hooks (useCallback, useMemo, useEffect) where declarative query expressions improve readability and reduce mutation risks
- TypeScript type inference works naturally with Array.find() and Map.get(), providing type safety without additional annotations

## Consequences

Positive:
- Improved code readability through declarative query expressions that clearly state intent
- Type-safe lookups with TypeScript inference for both Array.find() return types and Map.get() value types
- Reduced mutation bugs by avoiding manual index tracking and loop state management
- Consistent pattern across components simplifies code review and onboarding

Negative:
- Array.find() has O(n) complexity which may impact performance on large collections without indexing
- Predicate functions in Array.find() create additional function allocations unless memoized
- Map.get() returns undefined for missing keys, requiring explicit undefined checks or optional chaining
- Pattern may be overused in cases where direct indexed access or filter operations are more appropriate

## Alternatives

- Use manual for loops with break statements for array queries (rejected)
  Rejected because: Manual loops increase code verbosity, require explicit index management, and reduce type inference quality compared to Array.find()
  When valid: Performance-critical hot paths with profiled bottlenecks where loop overhead is measurable
- Use object property access instead of Map.get() for cache layers (rejected)
  Rejected because: Objects lack built-in size tracking, require hasOwnProperty checks, and have prototype chain lookup overhead; Maps provide cleaner semantics for dynamic key-value storage
  When valid: When keys are known statically and type definitions benefit from object shape inference
- Use lodash _.find() and _.get() utilities for collection queries (rejected)
  Rejected because: Native Array.find() and Map.get() provide equivalent functionality without external dependency overhead; lodash adds bundle size without benefit for these simple operations
  When valid: When project already depends on lodash for other utilities and consistency is prioritized

## Risks

- Performance degradation when Array.find() is used on large collections in render-critical paths
  Mitigation: Profile hot paths with React DevTools Profiler; convert to indexed Map structures for collections exceeding 100 items with frequent lookups
  Owner: Engineering team
- Undefined handling errors when Map.get() returns undefined for missing keys without proper checks
  Mitigation: Enable TypeScript strict null checks; use optional chaining (?.) or nullish coalescing (??) for Map.get() results; add ESLint rule for undefined checks
  Owner: Engineering team
- Memory leaks from Map collections that grow unbounded without cleanup via Map.delete()
  Mitigation: Implement cleanup in useEffect return functions; document Map lifecycle expectations; add size monitoring in development mode
  Owner: Engineering team

## Implementation Notes

- Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in effect cleanup functions
- When querying arrays of configuration objects (plugins, options), use Array.find() with arrow function predicates that compare identity or property values
- For DOM collection queries like document.styleSheets, wrap in Array.from() before applying Array.find() to convert StyleSheetList to array
- Combine Map.get() checks with early returns or default values using nullish coalescing to handle missing keys gracefully

## Continuation Context


Verify commands:
- grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l
- grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5
- grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

Accept when:
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites

## Enforcement

- Verified by: Code review checklist requiring Array.find() for array queries and Map.get() for keyed lookups
- Verified by: ESLint custom rule detecting manual loops over arrays with single-item return patterns
- Verified by: TypeScript strict mode compilation enforcing undefined checks on Map.get() results
- Violation handling: Code review feedback requesting refactor to Array.find() or Map.get() with rationale
- Violation handling: ESLint warnings in CI pipeline for detected manual iteration patterns
- Violation handling: Blocked PR merge if TypeScript strict null check violations exist
- Exception process: Document performance justification with profiling data in code comments
- Exception process: Request tech lead review with benchmark comparison showing measurable improvement
- Exception process: Add inline ESLint disable comment with ticket reference for tracking