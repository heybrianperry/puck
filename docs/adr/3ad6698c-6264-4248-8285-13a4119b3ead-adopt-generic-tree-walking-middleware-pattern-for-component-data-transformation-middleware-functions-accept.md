# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Middleware Functions Accept

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires traversal and transformation of hierarchical component data structures containing props, content, zones, and root elements
- State management operations need interception points to record history, trigger callbacks, and control action propagation through reducer middleware
- Generic type-parameterized functions enable reusable traversal logic across different data shapes while maintaining type safety
- The packages/core module exports public contracts (walkTree, StateReducer, createReducer) that establish middleware boundaries for data transformation and state interception

## Problem Statement

Component-based systems with nested data structures require consistent mechanisms to traverse, transform, and intercept operations without coupling traversal logic to specific data shapes or business logic, while maintaining type safety and enabling extensibility through callback-based middleware patterns.

## Decision

1. MUST: Middleware functions MUST accept callback functions that receive data items and contextual options (parentId, propName) and return transformed data or null

## Policy Block

- MUST Middleware functions MUST accept callback functions that receive data items and contextual options (parentId, propName) and return transformed data or null

In scope:
- Tree-walking operations on ComponentData, RootData, and UserData structures
- State reducer middleware for action interception and history recording
- Generic callback-based transformation functions in packages/core
- Public API contracts exported from core library modules

Out of scope:
- Business logic within callback functions (delegated to consumers)
- Specific component rendering implementations
- UI framework integration details beyond state management
- Non-core library modules outside packages/core

## Rationale

- The evidence shows two distinct middleware patterns: walkTree for data traversal and storeInterceptor for state management, both using generic type parameters and callback-based extension points
- Exporting public contracts (walkTree, StateReducer, createReducer) from packages/core establishes clear boundaries between library infrastructure and consumer code
- The pattern enables separation of traversal mechanics from transformation logic, allowing consumers to inject custom behavior without modifying core library code
- Type parameterization with constraints (T extends ComponentData | RootData) ensures compile-time safety while supporting diverse data shapes

## Consequences

Positive:
- Reusable traversal and interception logic reduces code duplication across different data transformation scenarios
- Type-safe generic functions prevent runtime errors from incompatible data shapes
- Callback-based middleware enables extensibility without modifying core library code
- Clear API boundaries (public contracts) facilitate testing and modular development

Negative:
- Generic type parameters with complex constraints increase cognitive load for developers unfamiliar with advanced TypeScript patterns
- Callback-based middleware can create debugging challenges when multiple transformations are chained
- The abstraction overhead may be excessive for simple, non-recursive data transformations
- Middleware interception points introduce potential performance bottlenecks in high-frequency state updates

## Alternatives

- Implement specific traversal functions for each data type (ComponentData, RootData, UserData) without generic type parameters (rejected)
  Rejected because: Would result in code duplication and loss of type safety across different data shapes, contradicting the observed pattern of generic reusability
  When valid: In codebases with only one or two fixed data structures where generic abstraction overhead outweighs benefits
- Use class-based visitor pattern with inheritance hierarchy instead of functional middleware (rejected)
  Rejected because: Evidence shows functional callback-based approach, not object-oriented visitor pattern; class-based approach would conflict with existing functional architecture
  When valid: In object-oriented codebases where visitor pattern is already established and inheritance is preferred over composition
- Embed transformation logic directly in data structures using methods rather than external middleware functions (rejected)
  Rejected because: Would couple data structures to transformation logic, violating separation of concerns evident in the middleware boundary pattern
  When valid: For simple data structures with fixed, unchanging transformation requirements

## Risks

- Complex generic type constraints may create steep learning curve for new contributors and increase onboarding time
  Mitigation: Provide comprehensive TypeScript documentation with examples, type aliases for common patterns, and inline comments explaining constraint rationale
  Owner: engineering team
- Middleware callback chains may introduce performance degradation in high-frequency state updates or deep tree traversals
  Mitigation: Implement performance benchmarks for walkTree and storeInterceptor, establish performance budgets, and consider memoization for expensive transformations
  Owner: engineering team
- Callback-based middleware can obscure control flow and make debugging difficult when multiple transformations are composed
  Mitigation: Add logging/tracing capabilities to middleware functions, provide development-mode warnings for common misuse patterns, and document callback execution order
  Owner: engineering team

## Implementation Notes

- When implementing tree-walking middleware, ensure callback functions handle null returns gracefully to support filtering operations
- State reducer interceptors should check action.recordHistory flag before invoking record callbacks to avoid unnecessary history entries
- Use mapFields or similar recursive utilities to process nested slot-based content within walkTree implementations
- Export middleware functions as public API contracts from core library index files to establish clear module boundaries

## Continuation Context


Verify commands:
- grep -r 'function walkTree<' packages/core --include='*.ts' | grep -q 'ComponentData\|RootData\|UserData'
- grep -r 'function storeInterceptor' packages/core --include='*.ts' | grep -q 'reducer.*onAction'
- grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core/index.ts packages/core/reducer/index.ts

Accept when:
- Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options
- State reducer middleware wraps base reducers and provides onAction hooks for interception
- Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules

## Enforcement

- Verified by: TypeScript compiler type checking for generic constraints and function signatures
- Verified by: Code review verification that middleware functions follow callback-based pattern
- Verified by: Automated grep-based checks in CI pipeline for public API exports
- Violation handling: TypeScript compilation failures block merge for type constraint violations
- Violation handling: Code review feedback requests refactoring for non-compliant middleware implementations
- Violation handling: CI pipeline failures for missing public API exports require correction before merge
- Exception process: Document architectural rationale for deviation in ADR amendment or new ADR
- Exception process: Obtain approval from core library maintainers for changes to public API contracts
- Exception process: Add inline comments explaining why standard middleware pattern cannot be applied