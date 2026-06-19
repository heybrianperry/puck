# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Components Use Set

These rules are ALWAYS ACTIVE for all React components in `packages/core/components/` that implement virtualized rendering, drag-drop interactions, or dynamic component lifecycles requiring explicit cleanup of measurement references and virtualizer handles.

### Rules

- **R-LIFECYCLE-001** MUST: Use Map data structures for component-specific resource registration (measurement callbacks, virtualizer handles, cached dimensions) when explicit cleanup is required on component unmount or ID change.
- **R-LIFECYCLE-002** MUST: Call `Map.delete(componentId)` in useEffect cleanup functions or component unmount handlers for every `Map.set(componentId, value)` operation.
- **R-LIFECYCLE-003** MUST: Include componentId (or equivalent unique key) in useEffect dependency arrays to ensure cleanup runs when component identifiers change, not just on unmount.
- **R-LIFECYCLE-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) when multiple components share a registry to prevent key collisions.
- **R-LIFECYCLE-005** MAY: Use Set data structures for simpler boolean membership tracking (e.g., `nextPinnedIndexes.add(currentIndex)`) when resource cleanup is not required.
- **R-LIFECYCLE-006** MUST: Store Map registries in `useRef().current` or module-level variables to persist across renders without causing re-renders.
- **R-LIFECYCLE-007** MUST: Verify `Map.get()` returns a value before using it to handle race conditions where cleanup may have already occurred.

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
- All `Map.set()` calls in component lifecycle code have corresponding `Map.delete()` calls in useEffect cleanup functions or component unmount handlers
- Grep verification shows componentId or equivalent keys are used consistently in `.get()`, `.set()`, and `.delete()` operations within the same component
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components
- ESLint custom rule verification passes with no Map.set() without corresponding delete() in useEffect cleanup

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map-based lifecycle management patterns MUST be verified before merge. Memory leak detection in integration tests MUST pass. Code review MUST confirm Map cleanup completeness and consistency.
</enforcement>