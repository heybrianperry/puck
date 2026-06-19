# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Components Verify Resource

These rules are ALWAYS ACTIVE for all React components in `packages/core/components/` that implement virtualized rendering, drag-drop interactions, or dynamic component lifecycles requiring explicit cleanup of measurement references and virtualizer handles.

### Rules

- **R-MAP-001** SHOULD: Components SHOULD verify resource existence with Map.get() before performing operations to handle race conditions during cleanup.
- **R-MAP-002** MUST: All Map.set() calls in component lifecycle code MUST have corresponding Map.delete() calls in useEffect cleanup functions or component unmount handlers.
- **R-MAP-003** MUST: Component IDs MUST be included in useEffect dependency arrays to ensure cleanup runs when IDs change, not just on unmount.
- **R-MAP-004** SHOULD: Compound keys (e.g., `${zoneId}:${componentId}`) SHOULD be used when multiple components share a registry to prevent key collisions.
- **R-MAP-005** MUST: Map registries MUST be stored in useRef().current or module-level variables to persist across renders without causing re-renders.

### Verify

```bash
# Find all Map.set() calls and verify corresponding cleanup
grep -r 'Map.*\.set(' packages/core/components/ | xargs -I {} sh -c 'echo {} && grep -A 20 "{}" | grep -E "(useEffect|delete)"'

# Verify .delete() calls use consistent component ID patterns
grep -r '\.delete(' packages/core/components/ --include='*.tsx' --include='*.ts' -B 5 | grep -E '(componentId|zoneCompound)'

# Run lifecycle and cleanup tests
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cleanup|unmount|lifecycle'
```

**Accept when:**
- All Map.set() calls in component lifecycle code have corresponding Map.delete() calls in useEffect cleanup functions or component unmount handlers.
- Grep verification shows componentId or equivalent keys are used consistently in .get(), .set(), and .delete() operations within the same component.
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components.
- ESLint custom rule verification passes with no Map.set() calls without corresponding cleanup in useEffect.

<enforcement>
Clause Code MUST NOT skip or defer verification of Map lifecycle management in components using virtualization or drag-drop patterns. Code review MUST verify Map cleanup completeness before merge. CI build MUST fail if ESLint detects Map.set() without cleanup in new or modified components.
</enforcement>