# Standardize Map-Based Service Definition Lifecycle Management: Query Parameter Based

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React-based UI components require dynamic registration and cleanup of runtime resources such as virtualizers, measurement references, and component-specific state
- The codebase uses Map data structures to associate component identifiers with their corresponding service handles, measurement data, and lifecycle-managed resources
- URLSearchParams-based configuration retrieval patterns appear in client-side rendering contexts where runtime behavior is controlled by query parameters
- Component lifecycle hooks (useEffect, useCallback) coordinate with Map-based registries to ensure proper resource allocation and deallocation during mount/unmount cycles

## Problem Statement

Without a consistent pattern for managing service definitions and their lifecycle, components risk resource leaks, stale references, and inconsistent cleanup behavior. The system needs a standardized approach to register, retrieve, and delete service definitions that integrates cleanly with React component lifecycles and supports both imperative (Map operations) and declarative (query parameter) configuration patterns.

## Decision

1. SHOULD: Query parameter-based service configuration SHOULD use URLSearchParams.get() for retrieval and validate presence before use

## Policy Block

- SHOULD Query parameter-based service configuration SHOULD use URLSearchParams.get() for retrieval and validate presence before use

## Rationale

- Map data structures provide O(1) lookup performance for service definitions while supporting dynamic registration and cleanup without array reallocation overhead
- Explicit delete operations prevent memory leaks in long-lived single-page applications where components mount and unmount repeatedly
- URLSearchParams.get() provides a standardized browser API for extracting runtime configuration from query strings, enabling feature flags and behavior toggles without code changes
- Integration with React lifecycle hooks ensures service definitions are managed in sync with component visibility and resource availability

## Consequences

Positive:
- Predictable resource cleanup reduces memory leaks and stale reference bugs in dynamic UI components
- Map-based lookups provide constant-time access to service definitions regardless of registry size
- Query parameter configuration enables runtime behavior changes without redeployment or code modification
- Consistent lifecycle management patterns improve code readability and reduce cognitive load for developers

Negative:
- Map-based registries require manual lifecycle management and are not automatically garbage collected until explicitly deleted
- Query parameter dependencies create implicit coupling between URL structure and component behavior
- Ref-based Map storage bypasses React's reactivity system, requiring manual synchronization for UI updates
- Multiple lifecycle management patterns (Map operations, query params) increase the surface area for implementation errors

## Alternatives

- Use React Context for service definition storage with automatic cleanup via provider unmounting (rejected)
  Rejected because: Context re-renders all consumers on updates, creating performance overhead for frequently updated service definitions like measurement caches
  When valid: When service definitions are read-only or update infrequently, and component tree structure naturally aligns with service boundaries
- Store service definitions in component state (useState) for automatic React lifecycle integration (rejected)
  Rejected because: State updates trigger re-renders, which is unnecessary overhead for imperative service registries that don't affect rendering
  When valid: When service definition changes must trigger UI updates or when the definition directly influences rendered output
- Use WeakMap for automatic garbage collection of service definitions when component keys are collected (deferred)
  Rejected because: WeakMap keys must be objects, requiring wrapper objects for string-based component IDs, and lack enumeration capabilities needed for debugging
  When valid: When component identifiers are already object references and automatic cleanup is more important than registry introspection

## Risks

- Forgotten delete operations in cleanup functions cause memory leaks as Map registries accumulate stale entries
  Mitigation: Implement ESLint rules to verify useEffect cleanup functions include corresponding delete calls for all set operations, and add runtime leak detection in development mode
  Owner: Engineering team
- Query parameter changes during component lifecycle may cause inconsistent behavior if not re-evaluated
  Mitigation: Document query parameter evaluation timing and consider useEffect dependencies on search params for dynamic reconfiguration scenarios
  Owner: Engineering team
- Map key collisions between components with identical IDs but different scopes cause service definition overwrites
  Mitigation: Use compound keys (e.g., zoneCompound) that include scope identifiers, and validate key uniqueness in development builds
  Owner: Engineering team

## Implementation Notes

- Wrap Map registries in useRef to prevent recreation on every render: const registryRef = useRef(new Map())
- Always pair set operations with delete operations in useEffect cleanup: useEffect(() => { map.set(id, value); return () => map.delete(id); }, [id])
- For query parameter configuration, extract params once at component mount and store in state if dynamic updates are needed
- Use compound keys for nested component hierarchies to prevent ID collisions: const key = `${zoneId}:${componentId}`

## Continuation Context


Verify commands:
- grep -r 'measuredItemHeights\.set\|measureRefsRef\.current\.set\|rootVirtualizers\.set' --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'measuredItemHeights\.delete\|measureRefsRef\.current\.delete\|rootVirtualizers\.delete' --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'params\.get(' --include='*.tsx' --include='*.ts' | grep -c 'URLSearchParams\|searchParams'

Accept when:
- The number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions
- All URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration
- Map-based registries are stored in useRef hooks rather than component state or module-level variables

## Enforcement

- Verified by: Static analysis via ESLint custom rules detecting Map operations without corresponding cleanup
- Verified by: Code review checklist items for lifecycle management patterns in React components
- Verified by: Runtime leak detection in development builds tracking Map size growth over component mount/unmount cycles
- Violation handling: ESLint errors block CI pipeline for Map set operations without paired delete in cleanup functions
- Violation handling: Code review rejection for components with Map-based registries lacking explicit lifecycle management
- Violation handling: Development console warnings when Map registries exceed size thresholds indicating potential leaks
- Exception process: Document justification for Map registries with intentional long-lived entries that should not be deleted
- Exception process: Add inline comments with LEAK-SAFE annotation explaining why cleanup is unnecessary for specific cases
- Exception process: Obtain architecture team approval for alternative lifecycle management patterns that deviate from Map-based approach