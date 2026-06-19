# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Middleware Interceptors Provide

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The packages/core library exports public API contracts (walkTree, StateReducer, createReducer) that operate on hierarchical component data structures with zones, content arrays, and nested props
- Tree-walking and state reduction operations require middleware-style interceptor functions that transform data while preserving type safety across generic user configurations
- The walkTree function in packages/core/lib/data/walk-tree.ts accepts a callback function that processes Content nodes at each level, enabling extensible transformation logic without modifying core traversal
- The storeInterceptor function in packages/core/reducer/index.ts wraps a StateReducer to intercept actions, record history selectively, and invoke onAction callbacks, separating cross-cutting concerns from reduction logic
- Both patterns use TypeScript generics (UserConfig, UserGenerics, UserData) to maintain type safety while allowing consumers to customize data shapes and behavior

## Problem Statement

Public APIs that expose tree-walking and state management operations need a mechanism to allow consumers to inject custom transformation and interception logic without coupling the core traversal or reduction algorithms to specific business logic, while maintaining type safety across diverse user-defined data structures.

## Decision

1. MAY: Middleware interceptors MAY provide conditional execution logic based on action metadata (recordHistory flag, action type filtering)

## Policy Block

- MAY Middleware interceptors MAY provide conditional execution logic based on action metadata (recordHistory flag, action type filtering)

In scope:
- Public API functions exported from packages/core that traverse or transform hierarchical data
- State reducer interceptors that wrap core reduction logic with cross-cutting concerns
- Tree-walking operations over ComponentData, RootData, zones, and content arrays
- Middleware functions that accept user-defined callbacks for data transformation

Out of scope:
- Internal utility functions not exposed as public API contracts
- Direct data mutations without callback-based transformation
- Non-hierarchical data processing operations
- Synchronous event handlers that do not intercept data flow

## Rationale

- The evidence shows walkTree and storeInterceptor both implement callback-based middleware patterns, with walkTree accepting a callbackFn that processes Content nodes and storeInterceptor wrapping reducers with record and onAction callbacks
- This pattern enables separation of concerns by isolating traversal/reduction logic from transformation logic, allowing the core library to remain generic while consumers inject domain-specific behavior
- TypeScript generics (UserConfig, UserGenerics<UserConfig>, G['UserData']) maintain type safety across the middleware boundary, preventing type erasure while supporting diverse consumer data shapes
- The pattern appears in 2 files with 90% confidence, indicating a deliberate architectural choice for public API design in the core package

## Consequences

Positive:
- Consumers can extend tree-walking and state management behavior without forking or modifying core library code
- Type safety is preserved across middleware boundaries through generic type parameters
- Cross-cutting concerns (history recording, action logging) are cleanly separated from core algorithms
- The callback pattern enables composable transformation pipelines where multiple middleware functions can be chained

Negative:
- Callback-based APIs increase cognitive complexity for consumers who must understand the execution context and callback signature
- Generic type parameters can produce verbose type signatures that are difficult to read and debug
- Middleware interception adds runtime overhead for every traversal or state transition
- Debugging becomes more difficult when transformation logic is distributed across multiple callback functions

## Alternatives

- Use class-based inheritance with protected methods that consumers override for custom behavior (rejected)
  Rejected because: Inheritance couples consumers to the core library's class hierarchy and prevents composition of multiple transformation behaviors, whereas callback-based middleware allows flexible composition
  When valid: When the transformation logic is tightly coupled to the core algorithm and a single extension point is sufficient
- Expose raw data structures and require consumers to implement their own traversal logic (rejected)
  Rejected because: This shifts the complexity of handling zones, content arrays, and nested props to every consumer, duplicating traversal logic and increasing the risk of bugs
  When valid: When the data structure is simple and flat, or when consumers need full control over traversal order and strategy
- Use a plugin system with registration and lifecycle hooks instead of direct callback parameters (deferred)
  Rejected because: null
  When valid: When the number of extension points grows beyond 2-3 callbacks, or when plugins need to coordinate with each other through shared state

## Risks

- Callback functions that mutate shared state or have side effects can introduce non-deterministic behavior and race conditions
  Mitigation: Document that callbacks should be pure functions or clearly document side effect expectations; consider adding runtime warnings for detected mutations
  Owner: engineering team
- Complex generic type constraints may produce cryptic TypeScript errors that are difficult for consumers to debug
  Mitigation: Provide concrete type examples in documentation; create type utility helpers that simplify common use cases; add integration tests demonstrating typical usage patterns
  Owner: engineering team
- Performance degradation when callbacks are invoked on large data trees with thousands of nodes
  Mitigation: Profile walkTree performance with realistic data sizes; consider adding memoization or short-circuit options; document performance characteristics in API documentation
  Owner: engineering team

## Implementation Notes

- When implementing tree-walking middleware, always provide context information (parentId, propName) to callbacks to enable location-aware transformations
- Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void
- For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback
- Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes)
- Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations

## Continuation Context


Verify commands:
- grep -r 'function.*<.*>.*callbackFn' packages/core/lib --include='*.ts' | grep -v test
- grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core --include='*.ts'
- tsc --noEmit packages/core/lib/data/walk-tree.ts packages/core/reducer/index.ts

Accept when:
- All public API functions that traverse hierarchical data accept callback parameters with context information
- TypeScript compilation succeeds without type errors for generic middleware functions
- Exported public API contracts (walkTree, StateReducer) are present in packages/core with callback-based signatures

## Enforcement

- Verified by: TypeScript compiler checks enforce callback signature contracts at build time
- Verified by: Code review verifies that new tree-walking or state management functions follow the callback-based middleware pattern
- Verified by: Integration tests validate that callbacks receive correct context information and return values are handled properly
- Violation handling: TypeScript compilation failures block merge for functions that violate generic type constraints
- Violation handling: Code review feedback requests refactoring of direct data mutations to use callback-based transformation
- Violation handling: Documentation updates are required when new middleware patterns are introduced that deviate from established conventions
- Exception process: Performance-critical paths may bypass callback overhead with explicit documentation of the tradeoff
- Exception process: Internal utility functions not exposed as public API are exempt from callback-based middleware requirements
- Exception process: Exceptions require architectural review and must document the rationale in code comments