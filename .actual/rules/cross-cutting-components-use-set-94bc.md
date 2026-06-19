# Standardize Service Boundary Definitions Using Map-Based Lifecycle Management: Components Use Set

These rules are ALWAYS ACTIVE for React components in packages/core/components that expose public API contracts, virtualization and drag-drop components managing multiple child instances, client-side applications using URL parameters for runtime configuration, and components using useCallback, useMemo, useEffect hooks with Map-based state.

### Rules

- **R-SBD-001** MAY: Components MAY use Set data structures (nextPinnedIndexes.add) for boundary coordination when tracking unique values without associated metadata.
- **R-SBD-002** MUST: All components using Map-based lifecycle management implement cleanup via Map.delete() in useEffect return functions.
- **R-SBD-003** MUST: No Map instances are directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports).
- **R-SBD-004** MUST: All URLSearchParams.get() calls provide default values or null checks to handle missing parameters.
- **R-SBD-005** SHOULD: Use useRef to store Map instances that persist across render cycles without triggering re-renders (e.g., measureRefsRef.current).
- **R-SBD-006** SHOULD: Document Map key generation strategy in component JSDoc to ensure consistent usage across team members.

### Verify

```bash
# Check for Map.delete() usage in cleanup functions
grep -r 'Map.*\.delete(' packages/core/components --include='*.tsx' | wc -l

# Verify params.get() calls have null checks or defaults
grep -r 'params\.get(' apps/demo --include='*.tsx' | grep -v 'params\.get([^)]*) *!==' && echo 'Found params.get without null check' || echo 'All params.get calls have null checks'

# Ensure no Map instances are exported in public API
grep -r 'export.*Map' packages/core/components --include='*.tsx' && echo 'WARNING: Map exported in public API' || echo 'No Map instances exported'
```

**Accept when:**
- All components using Map-based lifecycle management implement cleanup via Map.delete() in useEffect return functions
- No Map instances are directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports)
- All URLSearchParams.get() calls provide default values or null checks to handle missing parameters
- Components use useRef for Map storage to persist across render cycles without triggering re-renders
- Map key generation strategy is documented in component JSDoc

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if useEffect cleanup is missing for Map operations. CI pipeline MUST fail if Map instances are exported from public API modules. Memory profiling tests MUST flag components with unbounded Map growth.
</enforcement>