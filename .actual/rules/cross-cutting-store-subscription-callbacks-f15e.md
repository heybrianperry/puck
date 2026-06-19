# Use Set-Based Subscription Tracking for Reactive State Management: Store Subscription Callbacks

These rules are ALWAYS ACTIVE for all React components using custom store implementations with Set or Map-based subscription tracking, particularly in virtualized drag-and-drop contexts.

### Rules

- **R-STORE-001** MUST: Store subscription callbacks in ref objects (e.g., `measureRefsRef.current`) to avoid stale closures in React hooks.
- **R-STORE-002** MUST: Pair all `Map.set()` or `Set.add()` operations with corresponding `delete()` calls in `useEffect` cleanup functions.
- **R-STORE-003** MUST: Ensure all `store.subscribe()` calls return unsubscribe functions that are invoked in `useEffect` cleanup phases.
- **R-STORE-004** SHOULD: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-STORE-005** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.
- **R-STORE-006** SHOULD: Implement custom serialization helpers for logging Map and Set structures to aid debugging.

### Verify

```bash
# Check for Set.add() operations without corresponding delete() calls
grep -r '\.add(' packages/core/components/ | grep -v '\.delete(' | wc -l

# Verify subscribe() calls are paired with useEffect cleanup
grep -r 'subscribe(' packages/core/components/ | grep -E 'useEffect|cleanup'

# Audit all Map and Set usage in component files
grep -r 'Map\|Set' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All `Set.add()` and `Map.set()` operations have corresponding `delete()` calls in component cleanup or removal paths.
- All `store.subscribe()` calls return unsubscribe functions that are invoked in `useEffect` cleanup.
- Map and Set usage is documented with key format specifications in component interfaces or type definitions.
- No orphaned Map entries or failed lookups occur due to inconsistent compound key generation.
- ESLint custom rules pass without detecting missing cleanup operations.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Code review MUST block merge if Map/Set usage lacks documented key format or cleanup logic. CI pipeline MUST fail if ESLint rules detect missing cleanup operations.
</enforcement>