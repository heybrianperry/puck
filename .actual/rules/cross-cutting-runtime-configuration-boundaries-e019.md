# Standardize Service Boundary Definitions Using Map-Based Lifecycle Management: Runtime Configuration Boundaries

These rules are ALWAYS ACTIVE for React components in packages/core/components that expose public API contracts, virtualization and drag-drop components managing multiple child instances, client-side applications using URL parameters for runtime configuration, and components using useCallback, useMemo, useEffect hooks with Map-based state.

### Rules

- **R-SBD-001** SHOULD: Runtime configuration boundaries SHOULD use URLSearchParams.get() for feature toggles and synchronization flags in client-side applications.
- **R-SBD-002** MUST: All components using Map-based lifecycle management MUST implement cleanup via Map.delete() in useEffect return functions.
- **R-SBD-003** MUST: Map instances MUST NOT be directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports).
- **R-SBD-004** MUST: All URLSearchParams.get() calls MUST provide default values or null checks to handle missing parameters.
- **R-SBD-005** SHOULD: Use useRef to store Map instances that persist across render cycles without triggering re-renders (e.g., measureRefsRef.current).
- **R-SBD-006** SHOULD: Document Map key generation strategy in component JSDoc to ensure consistent usage across team members.

### Verify

```bash
# Verify Map.delete() cleanup implementations
grep -r 'Map.*\.delete(' packages/core/components --include='*.tsx' | wc -l

# Verify params.get() has null checks or defaults
grep -r 'params\.get(' apps/demo --include='*.tsx' | grep -v 'params\.get([^)]*) *!==' && echo 'Found params.get without null check' || echo 'All params.get calls have null checks'

# Verify no Map instances exported in public API
grep -r 'export.*Map' packages/core/components --include='*.tsx' && echo 'WARNING: Map exported in public API' || echo 'No Map instances exported'
```

**Accept when:**
- All components using Map-based lifecycle management implement cleanup via Map.delete() in useEffect return functions
- No Map instances are directly exported in public API contracts (VirtualizedDropZone, DragDropContext, Client exports)
- All URLSearchParams.get() calls provide default values or null checks to handle missing parameters
- Map key generation strategy is documented in component JSDoc

<enforcement>
Verified by: Code review checklist requiring Map.delete() in cleanup functions
Verified by: ESLint custom rule detecting Map exports in files with public API contracts
Verified by: Integration tests verifying component cleanup and memory stability
Violation handling: CI pipeline fails if Map instances are exported from public API modules
Violation handling: Code review blocks merge if useEffect cleanup is missing for Map operations
Violation handling: Memory profiling tests flag components with unbounded Map growth
Exception process: Component owner documents exception rationale in ADR-AUTO-EXC-NNN format
Exception process: Architecture review board approves exceptions for legacy migration cases (EXC-001)
Exception process: Exceptions require migration timeline and must be reviewed quarterly
Claude Code MUST NOT skip or defer verification.
</enforcement>