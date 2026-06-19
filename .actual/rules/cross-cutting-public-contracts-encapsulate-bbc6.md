# Standardize Service Boundary Definitions Using Map-Based Lifecycle Management: Public Contracts Encapsulate

These rules are ALWAYS ACTIVE for React components in packages/core/components that expose public API contracts, virtualization and drag-drop components managing multiple child instances, client-side applications using URL parameters for runtime configuration, and components using useCallback, useMemo, useEffect hooks with Map-based state.

### Rules

- **R-SBD-001** MUST: Public API contracts MUST encapsulate Map operations (get/set/delete) within component implementations and not expose Map instances directly to consumers.

### Verify

```bash
# Verify Map.delete() cleanup is implemented
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
Clause Code MUST NOT skip or defer verification. Violations block CI pipeline and code review merge.
</enforcement>