# Standardize Service Boundary Definitions Using Map-Based Lifecycle Management: Public Contracts Encapsulate

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages component lifecycle and state through Map-based data structures that track references, measurements, and virtualizers across component boundaries
- React components in packages/core/components expose public contracts (VirtualizedDropZone, DragDropContext, Client) that require coordinated state management between parent and child components
- Service boundaries are defined through get/set/delete operations on Map instances (measuredItemHeights, measureRefsRef.current, rootVirtualizers) that isolate component-specific state
- URL parameter parsing (params.get) establishes runtime configuration boundaries for iframe synchronization and feature toggles in demo applications

## Problem Statement

Components with complex lifecycle requirements need explicit service boundaries to manage stateful resources (measurements, virtualizers, configuration) without leaking implementation details across API boundaries or creating implicit coupling between consumers and internal state management mechanisms.

## Decision

1. MUST: Public API contracts MUST encapsulate Map operations (get/set/delete) within component implementations and not expose Map instances directly to consumers

## Policy Block

- MUST Public API contracts MUST encapsulate Map operations (get/set/delete) within component implementations and not expose Map instances directly to consumers

In scope:
- React components in packages/core/components that expose public API contracts
- Virtualization and drag-drop components managing multiple child instances
- Client-side applications using URL parameters for runtime configuration
- Components using useCallback, useMemo, useEffect hooks with Map-based state

Out of scope:
- Server-side API endpoints or HTTP service boundaries
- Database schema definitions or persistence layer boundaries
- Build-time configuration or static asset boundaries
- Third-party library internal implementations

Exceptions:
- EXC-001: Legacy components migrating from array-based to Map-based tracking may temporarily expose Map instances during refactoring

## Rationale

- Map-based lifecycle management provides O(1) lookup and deletion for component instances identified by componentId or zoneCompound keys, as evidenced by measuredItemHeights.get/set/delete patterns
- Encapsulating Map operations within public contracts (VirtualizedDropZone, DragDropContext) maintains API stability while allowing internal state management optimization
- URLSearchParams.get() pattern in demo applications establishes clear runtime configuration boundaries without requiring prop drilling or global state
- The pattern appears consistently across 4 files with 85.88% confidence, indicating established architectural convention rather than isolated implementation

## Consequences

Positive:
- Component state isolation prevents unintended coupling between parent and child lifecycle management
- Map-based tracking enables efficient cleanup through delete operations, reducing memory leaks in long-lived applications
- Public API contracts remain stable while internal state management can evolve independently
- URL parameter boundaries enable runtime feature toggles without code changes or redeployment

Negative:
- Map-based state management increases implementation complexity compared to simple array or object storage
- Debugging requires understanding Map lifecycle across multiple render cycles and ref updates
- URLSearchParams parsing adds runtime overhead and requires client-side JavaScript execution
- Pattern requires consistent componentId or key generation strategy across component boundaries

## Alternatives

- Use React Context to share state across component boundaries instead of Map-based ref storage (rejected)
  Rejected because: Context re-renders all consumers on state changes, whereas Map-based refs allow surgical updates to specific component instances without triggering parent re-renders
  When valid: Valid for global configuration that changes infrequently and affects all consumers uniformly
- Expose Map instances directly in public API contracts for maximum flexibility (rejected)
  Rejected because: Direct Map exposure couples consumers to internal implementation details and prevents future optimization or refactoring of state management strategy
  When valid: Valid only for internal utility functions not exposed as public API contracts
- Use component props for all configuration instead of URL parameter parsing (deferred)
  Rejected because: Not rejected; deferred for server-side rendered components where URL parameters are not available
  When valid: Valid for server-side rendering contexts or when configuration must be controlled programmatically by parent components

## Risks

- Memory leaks if components fail to call Map.delete() during cleanup, causing unbounded Map growth in long-lived applications
  Mitigation: Enforce useEffect cleanup functions that call delete operations; add memory profiling to CI pipeline
  Owner: Component library maintainers
- Inconsistent componentId generation across boundaries may cause Map key collisions or orphaned entries
  Mitigation: Establish componentId generation convention using compound keys (e.g., zoneCompound); document in component API guidelines
  Owner: Engineering team
- URL parameter parsing may fail or return unexpected values if query string format changes or is malformed
  Mitigation: Provide default values for all params.get() calls; validate parameter values before use; document expected URL format
  Owner: Application developers

## Implementation Notes

- Use useRef to store Map instances that persist across render cycles without triggering re-renders (e.g., measureRefsRef.current)
- Implement cleanup in useEffect return functions: useEffect(() => { return () => mapRef.current.delete(id); }, [id])
- For URL parameters, provide sensible defaults: const enabled = params.get('feature') !== 'false' handles missing parameters gracefully
- Document Map key generation strategy in component JSDoc to ensure consistent usage across team members

## Continuation Context


Verify commands:
- grep -r 'Map.*\.delete(' packages/core/components --include='*.tsx' | wc -l
- grep -r 'params\.get(' apps/demo --include='*.tsx' | grep -v 'params\.get([^)]*) *!==' && echo 'Found params.get without null check' || echo 'All params.get calls have null checks'
- grep -r 'export.*Map' packages/core/components --include='*.tsx' && echo 'WARNING: Map exported in public API' || echo 'No Map instances exported'

Accept when:
- All components using Map-based lifecycle management implement cleanup via Map.delete() in useEffect return functions
- No Map instances are directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports)
- All URLSearchParams.get() calls provide default values or null checks to handle missing parameters

## Enforcement

- Verified by: Code review checklist requiring Map.delete() in cleanup functions
- Verified by: ESLint custom rule detecting Map exports in files with public API contracts
- Verified by: Integration tests verifying component cleanup and memory stability
- Violation handling: CI pipeline fails if Map instances are exported from public API modules
- Violation handling: Code review blocks merge if useEffect cleanup is missing for Map operations
- Violation handling: Memory profiling tests flag components with unbounded Map growth
- Exception process: Component owner documents exception rationale in ADR-AUTO-EXC-NNN format
- Exception process: Architecture review board approves exceptions for legacy migration cases
- Exception process: Exceptions require migration timeline and must be reviewed quarterly