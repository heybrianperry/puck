# Adopt Map-Based Service Boundary Management for Component Lifecycle: Cleanup Operations Invoke

These rules are ALWAYS ACTIVE for all component files in `packages/core/components/` that use Map-based registries for lifecycle resource management, particularly those integrating @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-CLEANUP-001** MUST: Cleanup operations MUST invoke delete on the registry (e.g., `measuredItemHeights.delete(componentId)`, `rootVirtualizers.delete(zoneCompound)`) during component unmount or resource disposal.

### Verify

```bash
# Count delete operations on known registries
grep -r '\.delete(' packages/core/components/ | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)' | wc -l

# Count Map-based registry declarations
grep -r 'new Map<' packages/core/components/ | grep -E '(Height|Ref|Virtualizer)' | wc -l

# Run component lifecycle tests with coverage
npm test -- --testPathPattern='(VirtualizedDropZone|DragDropContext)' --coverage --collectCoverageFrom='**/components/**/*.tsx'
```

**Accept when:**
- All Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) have corresponding delete operations in component cleanup paths
- At least 2 distinct registry instances are detected across VirtualizedDropZone and DragDropContext components
- Test coverage for registry lifecycle operations (get/set/delete) exceeds 80% in affected component files

<enforcement>
Clause R-CLEANUP-001 verification is mandatory. ESLint custom rules MUST block CI pipeline violations. Code review approval is withheld until registry lifecycle patterns are documented. Runtime warnings in development builds MUST log to console with component stack traces. Exceptions require tech lead approval with documented rationale.
</enforcement>