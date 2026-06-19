# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Map Registries Stored

These rules are ALWAYS ACTIVE for React components in `packages/core/components/` that implement virtualized rendering, drag-drop interactions, or dynamic component lifecycles requiring explicit cleanup of measurement references and virtualizer handles.

### Rules

- **R-MAP-001** MUST: Store Map registries in `useRef().current` or module-level variables to persist across renders without triggering re-renders.
- **R-MAP-002** MUST: Call `Map.delete(componentId)` in useEffect cleanup functions or component unmount handlers for every `Map.set()` operation.
- **R-MAP-003** MUST: Include `componentId` (or equivalent identifier) in useEffect dependency arrays to ensure cleanup runs when component IDs change, not just on unmount.
- **R-MAP-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) when multiple components share a registry to prevent key collisions.
- **R-MAP-005** SHOULD: Verify `Map.get()` returns a value before using it to handle race conditions where cleanup may have already occurred.
- **R-MAP-006** MUST: Implement explicit lifecycle boundaries with `Map.get()`, `Map.set()`, and `Map.delete()` operations visible in code review.

### Verify

```bash
# Detect all Map.set() calls in component lifecycle code
grep -r 'Map.*\.set(' packages/core/components/ | xargs -I {} sh -c 'echo {} && grep -A 20 "{}" | grep -E "(useEffect|delete)"'

# Verify componentId or equivalent keys are used consistently in get/set/delete operations
grep -r '\.delete(' packages/core/components/ --include='*.tsx' --include='*.ts' -B 5 | grep -E '(componentId|zoneCompound)'

# Run lifecycle and cleanup tests for virtualized components
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cleanup|unmount|lifecycle'

# Check for Map.set() without corresponding delete() in useEffect cleanup
grep -r 'Map.*\.set(' packages/core/components/ --include='*.tsx' --include='*.ts' -A 30 | grep -B 30 'useEffect' | grep -v 'delete'
```

**Accept when:**
- All `Map.set()` calls in component lifecycle code have corresponding `Map.delete()` calls in useEffect cleanup functions or component unmount handlers.
- Grep verification shows `componentId` or equivalent keys are used consistently in `.get()`, `.set()`, and `.delete()` operations within the same component.
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components.
- ESLint custom rule verification passes with no Map.set() calls lacking cleanup in new or modified components.
- Code review checklist confirms Map lifecycle management is complete and consistent across all virtualized and drag-drop components.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based lifecycle management. All R-MAP rules are mandatory for components using virtualization or drag-drop interactions. Violations must be caught during code review and CI build checks before merge.
</enforcement>