# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Implementations Extend Walktreeoptions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains hierarchical component data structures with nested content, zones, and slots that require uniform traversal and transformation operations
- Direct manipulation of nested component trees leads to code duplication and inconsistent handling of different data shapes (ComponentData, RootData, UserData)
- Type-safe traversal of generic component structures requires a middleware abstraction that preserves type information while allowing custom transformation logic
- The walkTree function in packages/core/lib/data/walk-tree.ts provides a reusable pattern for applying callback functions to all content nodes within component hierarchies

## Problem Statement

Component-based systems with nested data structures (props, slots, zones, content arrays) require consistent traversal and transformation operations. Without a standardized middleware pattern, each transformation operation must manually handle the recursive structure, leading to duplicated traversal logic, inconsistent handling of edge cases, and difficulty maintaining type safety across generic user configurations.

## Decision

1. MAY: Implementations MAY extend the WalkTreeOptions interface to provide additional context specific to their transformation requirements

## Policy Block

- MAY Implementations MAY extend the WalkTreeOptions interface to provide additional context specific to their transformation requirements

## Rationale

- The walkTree function demonstrates a proven pattern for type-safe traversal of complex component hierarchies, as evidenced by its generic type parameters and handling of multiple data shapes
- Centralizing tree traversal logic in middleware reduces code duplication and ensures consistent handling of nested structures across the codebase
- The callback-based approach provides flexibility for diverse transformation operations while maintaining a uniform interface for tree navigation
- Integration with mapFields utility shows effective separation between structural traversal and field-level transformation logic

## Consequences

Positive:
- Eliminates duplicated tree traversal logic across transformation operations, reducing maintenance burden
- Preserves TypeScript type information through generic parameters, enabling compile-time type safety for component transformations
- Provides consistent handling of all structural variants (props, zones, content arrays) in a single reusable abstraction
- Enables context-aware transformations by passing parentId and propName to callback functions

Negative:
- Adds abstraction layer that may obscure direct data manipulation for developers unfamiliar with the middleware pattern
- Generic type parameters increase complexity of function signatures and may require explicit type annotations at call sites
- Callback-based approach introduces indirection that can make debugging transformation logic more difficult
- Performance overhead from function calls and type checking may impact operations on very large component trees

## Alternatives

- Direct recursive traversal in each transformation function without middleware abstraction (rejected)
  Rejected because: Leads to code duplication, inconsistent handling of data shapes, and difficulty maintaining type safety across multiple transformation operations
  When valid: For one-off transformations in isolated contexts where reusability is not a concern
- Visitor pattern with explicit visitor interface and accept methods on data structures (rejected)
  Rejected because: Requires modifying data structure definitions to add accept methods, increasing coupling and reducing flexibility for external transformations
  When valid: When data structures are fully controlled and visitor operations are well-defined and stable
- Iterator-based traversal returning a flat sequence of nodes for external processing (deferred)
  Rejected because: null
  When valid: For read-only operations that benefit from lazy evaluation or when transformation logic requires access to all nodes before processing

## Risks

- Callback functions that modify shared state or have side effects may introduce non-deterministic behavior when tree structure changes
  Mitigation: Document that callbacks should be pure functions returning transformed content rather than mutating external state; provide examples of correct usage patterns
  Owner: engineering team
- Generic type constraints may not adequately capture all valid data shapes, leading to runtime errors despite TypeScript compilation success
  Mitigation: Implement comprehensive test suite covering all data shape variants (ComponentData, RootData, UserData with zones); add runtime validation for critical invariants
  Owner: engineering team
- Performance degradation on deeply nested component trees due to recursive function calls and callback overhead
  Mitigation: Profile performance on representative data structures; consider iterative implementation or memoization for performance-critical paths; document performance characteristics
  Owner: engineering team

## Implementation Notes

- Import walkTree from packages/core/lib/data/walk-tree.ts and provide a callback function that receives content nodes with their context (parentId, propName)
- Ensure callback functions return the transformed content or null/void to preserve original values; avoid mutating the input content directly
- When working with UserData structures, remember that walkTree handles root, content array, and zones automatically—callback logic should focus on individual node transformations
- Leverage the generic type parameters to maintain type safety: specify UserConfig and UserGenerics types that match your component data model

## Continuation Context


Verify commands:
- grep -r 'function walkTree' packages/core/lib/data/ --include='*.ts'
- grep -r 'walkTree<' packages/ --include='*.ts' | head -20
- grep -r 'mapFields' packages/core/lib/data/walk-tree.ts

Accept when:
- The walkTree function is present in packages/core/lib/data/walk-tree.ts with generic type parameters extending ComponentData, RootData, or UserData
- The function accepts a callback with signature (data: Content, options: WalkTreeOptions) and delegates field traversal to mapFields utility
- Implementation handles all three structural variants: props-based ComponentData, root-level data, and UserData with zones

## Enforcement

- Verified by: Code review verification that new transformation operations use walkTree middleware rather than implementing custom traversal
- Verified by: TypeScript compilation checks ensuring generic type parameters are correctly specified at call sites
- Verified by: Unit tests validating that walkTree handles all data shape variants correctly
- Violation handling: Code review feedback requesting refactoring to use walkTree middleware for component tree transformations
- Violation handling: Architecture review for cases where direct traversal is proposed, requiring justification for deviation from standard pattern
- Exception process: Document performance requirements or structural constraints that prevent use of walkTree middleware
- Exception process: Obtain approval from core team maintainers for alternative traversal implementations
- Exception process: Add inline comments explaining why standard middleware pattern is not applicable