# Adopt Map-Based Service Boundary Management for Component Lifecycle: Registries Implemented React

These rules are ALWAYS ACTIVE for all files in `packages/core/components/` that implement virtualized drag-and-drop interactions using @tanstack/react-virtual and @dnd-kit libraries, specifically VirtualizedDropZone and DragDropContext components.

### Rules

- **R-MAP-001** MUST: Implement component lifecycle resource registries (measuredItemHeights, measureRefsRef, rootVirtualizers) as Map instances or React refs containing Map instances to enable O(1) lookup and cleanup operations.
- **R-MAP-002** MUST: Pair all `registry.set()` calls with corresponding `registry.delete()` calls in useEffect cleanup functions or component disposal paths to prevent memory leaks.
- **R-MAP-003** MUST: Validate `registry.get()` return values for undefined before dereferencing, as components may query registries before registration completes.
- **R-MAP-004** SHOULD: Use TypeScript generics to constrain Map key-value types (e.g., `Map<string, number>` for measuredItemHeights, `Map<string, VirtualizerHandle>` for rootVirtualizers).
- **R-MAP-005** SHOULD: Initialize Map registries at module scope or within stable React refs to persist across render cycles without triggering re-renders.
- **R-MAP-006** MAY: Implement registries as React refs (measureRefsRef.current) or direct Map instances depending on re-render requirements and component lifecycle coordination needs.

### Verify

```bash
# Count delete operations for registry cleanup
grep -r '\.delete(' packages/core/components/ | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)' | wc -l

# Count Map-based registry instantiations
grep -r 'new Map<' packages/core/components/ | grep -E '(Height|Ref|Virtualizer)' | wc -l

# Run component lifecycle tests with coverage
npm test -- --testPathPattern='(VirtualizedDropZone|DragDropContext)' --coverage --collectCoverageFrom='**/components/**/*.tsx'
```

**Accept when:**
- All Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) have corresponding delete operations in component cleanup paths
- At least 2 distinct registry instances are detected across VirtualizedDropZone and DragDropContext components
- Test coverage for registry lifecycle operations (get/set/delete) exceeds 80% in affected component files
- No ESLint violations for Map.set() calls without corresponding Map.delete() in useEffect cleanup functions
- Runtime monitoring shows no unbounded growth in Map registry sizes during development builds

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-MAP rules are mandatory for components using @tanstack/react-virtual or @dnd-kit. Violations block CI pipeline. Code review approval is withheld until registry lifecycle patterns are documented and cleanup functions are present. Runtime warnings in development builds must be addressed before merge.
</enforcement>