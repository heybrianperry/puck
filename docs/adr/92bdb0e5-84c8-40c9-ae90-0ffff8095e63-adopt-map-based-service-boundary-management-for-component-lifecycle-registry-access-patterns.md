# Adopt Map-Based Service Boundary Management for Component Lifecycle: Registry Access Patterns

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses @dnd-kit/react, @dnd-kit/dom, and @dnd-kit/abstract for drag-and-drop interactions across virtualized components in packages/core/components
- Component lifecycle management requires coordination between VirtualizedDropZone and DragDropContext through shared state stores and virtualizer handles
- Measured item heights and measure references must persist across render cycles while remaining scoped to specific component instances identified by componentId
- The @tanstack/react-virtual library provides virtualization capabilities that require external coordination for height measurement and reference management
- Service boundaries are established through Map-based registries (measuredItemHeights, measureRefsRef.current, rootVirtualizers) that enable get/set/delete operations for component lifecycle coordination

## Problem Statement

Components in a virtualized drag-and-drop system require a mechanism to register, retrieve, and clean up lifecycle-scoped resources (height measurements, DOM references, virtualizer handles) across multiple independent component instances without coupling their internal implementations or creating memory leaks from stale registrations.

## Decision

1. SHOULD: Registry access patterns SHOULD be encapsulated within useEffect cleanup functions or useCallback handlers to ensure proper lifecycle coordination

## Policy Block

- SHOULD Registry access patterns SHOULD be encapsulated within useEffect cleanup functions or useCallback handlers to ensure proper lifecycle coordination

In scope:
- VirtualizedDropZone component height measurement and DOM reference management
- DragDropContext virtualizer handle registration and cleanup
- Component-scoped resources requiring lifecycle coordination across render cycles
- Integration points between @tanstack/react-virtual and @dnd-kit libraries

Out of scope:
- Global application state management unrelated to component lifecycle
- Server-side service registries or microservice discovery patterns
- Database connection pooling or external service client management
- Static configuration or compile-time dependency injection

## Rationale

- The evidence shows consistent use of Map.get(), Map.set(), and Map.delete() operations across measuredItemHeights, measureRefsRef.current, and rootVirtualizers registries in both VirtualizedDropZone.tsx and DragDropContext/index.tsx
- Map-based registries provide O(1) lookup performance for component-scoped resources while maintaining clear ownership boundaries through componentId and zoneCompound keys
- The pattern enables coordination between @tanstack/react-virtual virtualization and @dnd-kit drag-and-drop libraries without tight coupling, as evidenced by separate registry management in distinct component files
- Explicit delete operations in cleanup paths demonstrate intentional memory management and lifecycle awareness required for long-lived virtualized component trees

## Consequences

Positive:
- Clear service boundaries enable independent component lifecycle management without cross-component coupling
- Map-based registries provide predictable O(1) performance for resource lookup and cleanup operations
- Explicit delete operations prevent memory leaks in long-lived virtualized component hierarchies
- Pattern supports integration between third-party libraries (@tanstack/react-virtual, @dnd-kit) through neutral coordination layer

Negative:
- Manual registry management increases cognitive overhead compared to automatic garbage collection patterns
- Missing cleanup calls can lead to memory leaks that are difficult to detect without explicit monitoring
- Map-based registries lack type safety for key-value relationships without additional TypeScript constraints
- Debugging registry state requires runtime inspection tools rather than declarative component tree visualization

## Alternatives

- Use React Context to propagate lifecycle resources directly through component tree (rejected)
  Rejected because: Context propagation couples component hierarchy to resource access patterns and does not provide efficient cleanup mechanisms for component-scoped resources across virtualized lists with dynamic item counts
  When valid: When component hierarchy is shallow and static, and resource cleanup is managed by parent component unmount
- Implement WeakMap-based registries for automatic garbage collection (rejected)
  Rejected because: WeakMap keys must be objects and do not support primitive componentId strings; automatic GC timing is non-deterministic and incompatible with synchronous virtualizer height calculations
  When valid: When registry keys are object references and cleanup timing can be non-deterministic
- Centralize all lifecycle resources in a single Redux or Zustand store (deferred)
  Rejected because: Not rejected; pattern may evolve toward centralized store as evidenced by existing '../../store' imports, but current Map-based approach provides sufficient isolation
  When valid: When cross-component coordination requires time-travel debugging, persistence, or complex derived state computations

## Risks

- Missing delete calls in component cleanup paths cause memory leaks in long-running applications with frequent component mount/unmount cycles
  Mitigation: Implement ESLint rules to enforce delete operations in useEffect cleanup functions; add runtime monitoring to detect registry size growth anomalies
  Owner: engineering team
- Registry key collisions occur if componentId or zoneCompound values are not globally unique across component instances
  Mitigation: Enforce UUID or symbol-based key generation for component identifiers; add runtime assertions to detect duplicate registrations
  Owner: engineering team
- Race conditions emerge when multiple components concurrently access shared registries without synchronization
  Mitigation: Document registry access patterns as synchronous-only; use React's batching guarantees within useEffect and useCallback to ensure sequential updates
  Owner: engineering team

## Implementation Notes

- Initialize Map registries at module scope or within stable React refs (measureRefsRef.current) to persist across render cycles
- Always pair registry.set() calls with corresponding registry.delete() calls in useEffect cleanup functions or component disposal paths
- Validate registry.get() return values for undefined before dereferencing, as components may query before registration completes
- Use TypeScript generics to constrain Map key-value types: Map<string, number> for measuredItemHeights, Map<string, VirtualizerHandle> for rootVirtualizers

## Continuation Context


Verify commands:
- grep -r '\.delete(' packages/core/components/ | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)' | wc -l
- grep -r 'new Map<' packages/core/components/ | grep -E '(Height|Ref|Virtualizer)' | wc -l
- npm test -- --testPathPattern='(VirtualizedDropZone|DragDropContext)' --coverage --collectCoverageFrom='**/components/**/*.tsx'

Accept when:
- All Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) have corresponding delete operations in component cleanup paths
- At least 2 distinct registry instances are detected across VirtualizedDropZone and DragDropContext components
- Test coverage for registry lifecycle operations (get/set/delete) exceeds 80% in affected component files

## Enforcement

- Verified by: ESLint custom rule to detect Map.set() calls without corresponding Map.delete() in useEffect cleanup
- Verified by: Code review checklist item for registry lifecycle management in components using @tanstack/react-virtual or @dnd-kit
- Verified by: Runtime monitoring of Map registry sizes in development builds with warnings for unbounded growth
- Violation handling: ESLint violations block CI pipeline until cleanup functions are added
- Violation handling: Code review approval withheld until registry lifecycle patterns are documented
- Violation handling: Runtime warnings in development builds logged to console with component stack traces
- Exception process: Document exception rationale in code comments explaining why cleanup is handled externally
- Exception process: Add explicit test coverage demonstrating memory safety without standard cleanup pattern
- Exception process: Obtain approval from tech lead with justification for alternative lifecycle management approach