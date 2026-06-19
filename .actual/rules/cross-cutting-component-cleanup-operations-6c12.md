# Standardize Service Boundary Definitions Using Map-Based Lifecycle Management: Component Cleanup Operations

These rules are ALWAYS ACTIVE for React components in packages/core/components that expose public API contracts, virtualization and drag-drop components managing multiple child instances, client-side applications using URL parameters for runtime configuration, and components using useCallback, useMemo, useEffect hooks with Map-based state.

### Rules

- **R-CLEANUP-001** MUST: Component cleanup operations MUST use Map.delete() to remove tracked references when components unmount or identifiers become invalid.

### Verify

```bash
# Verify Map.delete() usage in component cleanup
grep -r 'Map.*\.delete(' packages/core/components --include='*.tsx' | wc -l

# Verify params.get() calls have null checks or defaults
grep -r 'params\.get(' apps/demo --include='*.tsx' | grep -v 'params\.get([^)]*) *!==' && echo 'Found params.get without null check' || echo 'All params.get calls have null checks'

# Verify no Map instances are exported in public API
grep -r 'export.*Map' packages/core/components --include='*.tsx' && echo 'WARNING: Map exported in public API' || echo 'No Map instances exported'
```

**Accept when:**
- All components using Map-based lifecycle management implement cleanup via Map.delete() in useEffect return functions
- No Map instances are directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports)
- All URLSearchParams.get() calls provide default values or null checks to handle missing parameters

<enforcement>
Clause R-CLEANUP-001 verification is mandatory. Code review MUST block merges if useEffect cleanup is missing for Map operations. CI pipeline MUST fail if Map instances are exported from public API modules. Memory profiling tests MUST flag components with unbounded Map growth.
</enforcement>