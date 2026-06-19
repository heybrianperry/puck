# Adopt Map-Based Service Boundary Management for Component Lifecycle: Registry Access Patterns

These rules are ALWAYS ACTIVE for all files in `packages/core/components/` that use Map-based registries for component lifecycle coordination, particularly VirtualizedDropZone, DragDropContext, and components integrating @tanstack/react-virtual or @dnd-kit libraries.

### Rules

- **R-REG-001** MUST: Initialize Map registries at module scope or within stable React refs (measureRefsRef.current) to persist across render cycles.
- **R-REG-002** MUST: Always pair registry.set() calls with corresponding registry.delete() calls in useEffect cleanup functions or component disposal paths.
- **R-REG-003** MUST: Validate registry.get() return values for undefined before dereferencing, as components may query before registration completes.
- **R-REG-004** SHOULD: Registry access patterns SHOULD be encapsulated within useEffect cleanup functions or useCallback handlers to ensure proper lifecycle coordination.
- **R-REG-005** SHOULD: Use TypeScript generics to constrain Map key-value types (e.g., Map<string, number> for measuredItemHeights, Map<string, VirtualizerHandle> for rootVirtualizers).
- **R-REG-006** SHOULD: Enforce UUID or symbol-based key generation for component identifiers to prevent registry key collisions across component instances.

### Verify

```bash
# Count delete operations on registry Maps
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
- Code review checklist confirms registry lifecycle management documentation for all components using @tanstack/react-virtual or @dnd-kit.

<enforcement>
Claude Code MUST NOT skip or defer verification. All Map-based registry access patterns MUST comply with R-REG-001 through R-REG-006. Violations block CI pipeline until cleanup functions are added and code review approval is obtained.
</enforcement>