# Adopt Map-Based Service Boundary Management for Component Lifecycle: Component Lifecycle Resources

These rules are ALWAYS ACTIVE for component lifecycle resource management in virtualized drag-and-drop systems, specifically for VirtualizedDropZone, DragDropContext, and related components using @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-CLR-001** MUST: Component lifecycle resources MUST support get, set, and delete operations through the Map-based registry interface.
- **R-CLR-002** MUST: All Map-based registries (measuredItemHeights, measureRefsRef.current, rootVirtualizers) MUST have corresponding delete operations in component cleanup paths.
- **R-CLR-003** MUST: Registry.set() calls MUST be paired with corresponding registry.delete() calls in useEffect cleanup functions or component disposal paths.
- **R-CLR-004** SHOULD: Validate registry.get() return values for undefined before dereferencing, as components may query before registration completes.
- **R-CLR-005** SHOULD: Use TypeScript generics to constrain Map key-value types (e.g., Map<string, number> for measuredItemHeights, Map<string, VirtualizerHandle> for rootVirtualizers).
- **R-CLR-006** SHOULD: Initialize Map registries at module scope or within stable React refs to persist across render cycles.
- **R-CLR-007** SHOULD: Enforce UUID or symbol-based key generation for component identifiers to prevent registry key collisions.

### Verify

```bash
# Count delete operations on lifecycle registries
grep -r '\.delete(' packages/core/components/ | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)' | wc -l

# Count Map-based registry declarations
grep -r 'new Map<' packages/core/components/ | grep -E '(Height|Ref|Virtualizer)' | wc -l

# Run component lifecycle tests with coverage
npm test -- --testPathPattern='(VirtualizedDropZone|DragDropContext)' --coverage --collectCoverageFrom='**/components/**/*.tsx'
```

**Accept when:**
- All Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) have corresponding delete operations in component cleanup paths.
- At least 2 distinct registry instances are detected across VirtualizedDropZone and DragDropContext components.
- Test coverage for registry lifecycle operations (get/set/delete) exceeds 80% in affected component files.
- ESLint custom rule detects no Map.set() calls without corresponding Map.delete() in useEffect cleanup.
- No runtime warnings for unbounded Map registry growth in development builds.

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CLR rules are mandatory for component lifecycle resource management. Violations block CI pipeline until cleanup functions are added and code review approval is obtained.
</enforcement>