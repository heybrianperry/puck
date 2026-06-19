# Adopt Map-Based Service Boundary Management for Component Lifecycle: Service Boundary Registries

These rules are ALWAYS ACTIVE for all files in `packages/core/components/` that implement virtualized drag-and-drop interactions using @tanstack/react-virtual and @dnd-kit libraries, specifically VirtualizedDropZone and DragDropContext components and their lifecycle resource management.

### Rules

- **R-SBR-001** MUST: Service boundary registries MUST use Map data structures with component-scoped keys (componentId, zoneCompound) for lifecycle resource management.
- **R-SBR-002** MUST: All Map.set() operations for lifecycle resources (measuredItemHeights, measureRefsRef, rootVirtualizers) MUST have corresponding Map.delete() calls in useEffect cleanup functions or component disposal paths.
- **R-SBR-003** MUST: Registry.get() return values MUST be validated for undefined before dereferencing, as components may query before registration completes.
- **R-SBR-004** SHOULD: Use TypeScript generics to constrain Map key-value types (e.g., Map<string, number> for measuredItemHeights, Map<string, VirtualizerHandle> for rootVirtualizers).
- **R-SBR-005** SHOULD: Initialize Map registries at module scope or within stable React refs to persist across render cycles.
- **R-SBR-006** MAY: Document registry access patterns as synchronous-only and use React's batching guarantees within useEffect and useCallback to ensure sequential updates.

### Verify

```bash
# Count delete operations for lifecycle registries
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
Claude Code MUST NOT skip or defer verification. All Map-based registries must be verified to have cleanup operations before accepting changes. ESLint violations block CI pipeline. Code review approval is withheld until registry lifecycle patterns are documented. Runtime warnings in development builds must be addressed.
</enforcement>