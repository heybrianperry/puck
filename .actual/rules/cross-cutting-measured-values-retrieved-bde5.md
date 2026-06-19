# Adopt Map-Based Service Boundary Management for Component Lifecycle: Measured Values Retrieved

These rules are ALWAYS ACTIVE for all files in `packages/core/components/` that use Map-based registries for component lifecycle coordination, particularly VirtualizedDropZone and DragDropContext implementations integrating @tanstack/react-virtual and @dnd-kit libraries.

### Rules

- **R-MAP-001** MUST: Validate measured values retrieved from registries for existence before use (e.g., `measuredItemHeights.get(componentId)` may return `undefined`).
- **R-MAP-002** MUST: Pair all `registry.set()` calls with corresponding `registry.delete()` calls in `useEffect` cleanup functions or component disposal paths.
- **R-MAP-003** MUST: Initialize Map registries at module scope or within stable React refs (e.g., `measureRefsRef.current`) to persist across render cycles.
- **R-MAP-004** SHOULD: Use TypeScript generics to constrain Map key-value types (e.g., `Map<string, number>` for measuredItemHeights, `Map<string, VirtualizerHandle>` for rootVirtualizers).
- **R-MAP-005** SHOULD: Enforce UUID or symbol-based key generation for component identifiers to prevent registry key collisions across component instances.
- **R-MAP-006** SHOULD: Document registry access patterns as synchronous-only and use React's batching guarantees within `useEffect` and `useCallback` to ensure sequential updates.

### Verify

```bash
# Count delete operations for Map-based registries
grep -r '\.delete(' packages/core/components/ | grep -E '(measuredItemHeights|measureRefsRef|rootVirtualizers)' | wc -l

# Count Map registry instantiations
grep -r 'new Map<' packages/core/components/ | grep -E '(Height|Ref|Virtualizer)' | wc -l

# Run component lifecycle tests with coverage
npm test -- --testPathPattern='(VirtualizedDropZone|DragDropContext)' --coverage --collectCoverageFrom='**/components/**/*.tsx'
```

**Accept when:**
- All Map-based registries (measuredItemHeights, measureRefsRef, rootVirtualizers) have corresponding delete operations in component cleanup paths.
- At least 2 distinct registry instances are detected across VirtualizedDropZone and DragDropContext components.
- Test coverage for registry lifecycle operations (get/set/delete) exceeds 80% in affected component files.
- No undefined dereferences occur on values retrieved from `registry.get()` calls without prior existence validation.

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint violations block CI pipeline until cleanup functions are added. Code review approval is withheld until registry lifecycle patterns are documented. Runtime warnings in development builds must be logged to console with component stack traces for unbounded registry growth.
</enforcement>