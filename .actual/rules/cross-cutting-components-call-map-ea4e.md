# Adopt Map-Based Service Lifecycle Management for Component Cleanup: Components Call Map

These rules are ALWAYS ACTIVE for all React components in `packages/core/components/` that use virtualization (@tanstack/react-virtual), drag-drop (@dnd-kit/react), or Map-based service registries for lifecycle management.

### Rules

- **R-MAP-001** MUST: Components MUST call `Map.delete(componentId)` in cleanup functions (useEffect return, component unmount) to remove registered resources when the component or its ID changes.

### Verify

```bash
# Find all Map.set() calls and verify corresponding delete() in cleanup
grep -r 'Map.*\.set(' packages/core/components/ | xargs -I {} sh -c 'echo {} && grep -A 20 "{}" | grep -E "(useEffect|delete)"'

# Verify componentId or equivalent keys are used consistently in get/set/delete
grep -r '\.delete(' packages/core/components/ --include='*.tsx' --include='*.ts' -B 5 | grep -E '(componentId|zoneCompound)'

# Run lifecycle and cleanup tests
npm test -- --testPathPattern='VirtualizedDropZone|DragDropContext' --testNamePattern='cleanup|unmount|lifecycle'
```

**Accept when:**
- All `Map.set()` calls in component lifecycle code have corresponding `Map.delete()` calls in useEffect cleanup functions or component unmount handlers
- Grep verification shows `componentId` or equivalent keys are used consistently in `.get()`, `.set()`, and `.delete()` operations within the same component
- Memory profiling tests confirm no accumulation of Map entries after repeated mount/unmount cycles of virtualized components

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map-based lifecycle management MUST include explicit cleanup. ESLint rules and memory profiling tests MUST pass before merge. Code review MUST verify Map cleanup completeness.
</enforcement>