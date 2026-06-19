# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Component Lifecycle Managers

These rules are ALWAYS ACTIVE for React components in `packages/core/components/` that implement virtualized rendering, drag-drop interactions, or dynamic component lifecycles requiring explicit cleanup of measurement references and virtualizer handles.

### Rules

- **R-LIFECYCLE-001** MUST: Component lifecycle managers MUST use Map data structures keyed by component ID for storing instance-specific resources such as measurement callbacks, virtualizer handles, or cached dimensions.
- **R-LIFECYCLE-002** MUST: All Map.set() calls in component lifecycle code MUST have corresponding Map.delete() calls in useEffect cleanup functions or component unmount handlers.
- **R-LIFECYCLE-003** MUST: Component ID MUST be included in useEffect dependency arrays to ensure cleanup runs when IDs change, not just on unmount.
- **R-LIFECYCLE-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) when multiple components share a registry to prevent key collisions.
- **R-LIFECYCLE-005** SHOULD: Store Map registries in useRef().current or module-level variables to persist across renders without causing re-renders.
- **R-LIFECYCLE-006** SHOULD: Verify Map.get() returns a value before using it to handle race conditions where cleanup may have already occurred.

### Verify

```bash
# Detect Map.set() calls and verify corresponding delete() in cleanup
grep -r 'Map.*\.set(' packages/core/components/ | xargs -I {} sh -c 'echo {} && grep -A 20 "{}" | grep -E "(useEffect|delete)"'

# Verify componentId or equivalent keys are used consistently in get/set/delete
grep -r '\.delete(' packages/core/components/ --include='*.tsx' --include='*.ts' -B 5 | grep -E '(componentId|zoneCompound)'

# Run lifecycle and cleanup tests
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cleanup|unmount|lifecycle'
```

**Accept when:**
- All Map.set() calls in component lifecycle code have corresponding Map.delete() calls in useEffect cleanup functions or component unmount handlers.
- Grep verification shows componentId or equivalent keys are used consistently in .get(), .set(), and .delete() operations within the same component.
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components.
- ESLint custom rule verification passes with no Map.set() without cleanup detected in new or modified components.

<enforcement>
Clause Code MUST NOT skip or defer verification of Map lifecycle management in components using virtualization or drag-drop. Code review MUST check for Map cleanup completeness. CI build MUST fail if ESLint detects Map.set() without cleanup. Memory leak detection in integration tests MUST pass before release.
</enforcement>