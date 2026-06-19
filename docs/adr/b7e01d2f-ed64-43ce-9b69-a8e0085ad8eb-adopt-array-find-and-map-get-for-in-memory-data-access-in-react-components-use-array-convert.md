# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Use Array Convert

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- React components in the core package require efficient in-memory data access patterns for managing virtualized lists, drag-drop state, and style sheet references
- The codebase uses Map data structures for caching measured item heights, virtualizer handles, and component references that require frequent lookups by identifier
- Array.find() operations are used to locate specific items in collections such as drag paths, zoom options, plugins, and style sheets based on predicate functions
- Performance-sensitive UI interactions (virtualization, drag-drop) necessitate O(1) Map lookups rather than O(n) array iterations for frequently accessed cached data
- The pattern emerged across 5 files with 85.76% confidence, indicating consistent architectural approach to data access in React component state management

## Problem Statement

React components require consistent, performant patterns for accessing in-memory data structures including cached measurements, component references, configuration options, and DOM elements. Without standardized data access patterns, components may use inefficient lookups, inconsistent APIs, or fail to leverage appropriate data structures for their access patterns, leading to performance degradation in interactive UI features.

## Decision

1. MUST: Use Array.from() to convert DOM collections (e.g., document.styleSheets) to arrays before applying find() operations

## Policy Block

- MUST Use Array.from() to convert DOM collections (e.g., document.styleSheets) to arrays before applying find() operations

In scope:
- React components managing virtualized lists with measured item heights
- Drag-drop context components maintaining virtualizer handles and zone state
- Components caching DOM references or measurement data by component ID
- Configuration lookup operations (zoom options, plugins, style sheets)
- Performance-sensitive UI interaction handlers using useCallback and useMemo

Out of scope:
- Server-side data access patterns
- Database query operations
- Network request caching
- File system operations
- External API integrations

## Rationale

- Evidence shows consistent use of Map.get() for cache lookups (measuredItemHeights.get(componentId), measureRefsRef.current.get(componentId), rootVirtualizers) across virtualization and drag-drop components
- Array.find() pattern appears in 4 of 5 files for searching collections with predicates, indicating established convention for configuration and DOM element lookups
- Map data structures provide O(1) access time for cached measurements and references, critical for maintaining 60fps performance in virtualized scrolling and drag operations
- The pattern separates concerns: Maps for keyed cache access, Arrays for predicate-based searches, aligning data structure choice with access pattern requirements

## Consequences

Positive:
- O(1) lookup performance for cached measurements and component references in performance-critical rendering paths
- Consistent API surface across components for data access operations (Map.get/set/delete, Array.find)
- Clear separation between keyed access patterns (Map) and predicate-based searches (Array.find)
- Reduced cognitive load for developers through predictable data access patterns

Negative:
- Map structures require explicit lifecycle management (set/delete) compared to simpler array operations
- Mixed use of Map and Array requires developers to understand when each is appropriate
- No type-safe guarantees that Map keys exist, requiring null checks on get() results
- Memory overhead of Map structures compared to arrays for small datasets

## Alternatives

- Use only arrays with Array.find() for all data access patterns (rejected)
  Rejected because: O(n) lookup performance is unacceptable for frequently accessed cached data in virtualized lists and drag-drop operations where 60fps performance is required
  When valid: Acceptable for small collections (< 10 items) accessed infrequently or configuration lookups not in hot paths
- Use object literals with bracket notation for keyed access instead of Map (rejected)
  Rejected because: Objects lack built-in delete semantics, iteration order guarantees, and size properties that Map provides; evidence shows Map is already established pattern
  When valid: Valid for static configuration objects that never change after initialization
- Implement custom cache abstraction layer wrapping Map operations (deferred)
  Rejected because: No evidence of abstraction need in current codebase; native Map API is sufficient and well-understood
  When valid: Consider if cache invalidation strategies, TTL, or LRU eviction become requirements

## Risks

- Memory leaks if Map entries are not properly deleted when components unmount or items are removed
  Mitigation: Implement cleanup in useEffect return functions; verify Map.delete() calls in component lifecycle; add memory profiling to CI
  Owner: Frontend Engineering Team
- Performance degradation if Array.find() is used in hot paths with large collections
  Mitigation: Profile render performance; establish linting rules to flag Array.find() in useCallback/useMemo with large arrays; document O(n) complexity
  Owner: Frontend Engineering Team
- Inconsistent null handling when Map.get() returns undefined for missing keys
  Mitigation: Establish convention for default values; use optional chaining; add TypeScript strict null checks; document expected behavior
  Owner: Frontend Engineering Team

## Implementation Notes

- Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders
- Use Map.get() for cache lookups in render paths and event handlers; always handle undefined return values
- Clean up Map entries in useEffect cleanup functions: measureRefsRef.current.delete(componentId), rootVirtualizers.delete(zoneCompound)
- Reserve Array.find() for configuration lookups (plugins, zoom options) and DOM collection searches where predicates are necessary
- Consider Map.has() before Map.get() when existence check is semantically distinct from value retrieval

## Continuation Context


Verify commands:
- grep -r 'Map.*\.get(' packages/core/components/ | wc -l
- grep -r '\.find(' packages/core/components/ | wc -l
- grep -r 'Map.*\.delete(' packages/core/components/ | grep -c 'useEffect\|cleanup'

Accept when:
- Map.get() usage count is greater than 10 across core components, indicating established pattern
- Array.find() usage is present in at least 3 component files for configuration/collection searches
- Map.delete() calls appear in useEffect cleanup contexts, demonstrating proper lifecycle management

## Enforcement

- Verified by: Code review checklist verifying Map usage for caches and Array.find for searches
- Verified by: ESLint custom rules flagging Array.find() in performance-critical paths with large collections
- Verified by: Performance profiling in CI measuring render times for virtualized components
- Violation handling: Code review feedback requesting refactor from array to Map for keyed cache access
- Violation handling: Performance regression alerts trigger investigation of data access patterns
- Violation handling: ESLint warnings require justification comment or refactor before merge
- Exception process: Document rationale in code comment explaining why alternative pattern is used
- Exception process: Obtain approval from frontend tech lead for exceptions in performance-critical paths
- Exception process: Add performance benchmark demonstrating acceptable performance characteristics