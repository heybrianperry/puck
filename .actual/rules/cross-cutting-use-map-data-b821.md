# Use Set-Based Subscription Tracking for Reactive State Management: Use Map Data

These rules are ALWAYS ACTIVE for all React components managing virtualized drag-and-drop interactions, state subscriptions, and dynamic collections with measured heights or pinned indexes.

### Rules

- **R-MAP-001** MUST: Use Map data structures (e.g., `measuredItemHeights.get/set/delete`, `rootVirtualizers.set/delete`) for key-value associations where components or items are indexed by compound identifiers.
- **R-MAP-002** MUST: Pair all `Map.set()` or `Set.add()` operations with corresponding `delete()` calls in `useEffect` cleanup functions or component removal paths.
- **R-MAP-003** MUST: Ensure all `store.subscribe()` calls return unsubscribe functions that are invoked in `useEffect` cleanup phases.
- **R-MAP-004** SHOULD: Store Map and Set references in `useRef` hooks to maintain stable references across React re-renders without triggering effects.
- **R-MAP-005** SHOULD: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-MAP-006** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.
- **R-MAP-007** MAY: Implement custom serialization helpers for logging and use Map/Set-aware debugging utilities when debugging non-serializable structures.

### Verify

```bash
# Check for Set.add() operations without corresponding delete() calls
grep -r '\.add(' packages/core/components/ | grep -v '\.delete(' | wc -l

# Verify store.subscribe() calls are paired with cleanup in useEffect
grep -r 'subscribe(' packages/core/components/ | grep -E 'useEffect|cleanup'

# Audit all Map and Set usage across component files
grep -r 'Map\|Set' packages/core/components/ --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All `Set.add()` and `Map.set()` operations have corresponding `delete()` calls in component cleanup or removal paths.
- All `store.subscribe()` calls return unsubscribe functions that are invoked in `useEffect` cleanup.
- Map and Set usage is documented with key format specifications in component interfaces or type definitions.
- ESLint custom rules detect and flag `Set.add/Map.set` without corresponding `delete` in the same component.
- Unit tests assert that subscription cleanup functions are called on component unmount.

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. Code review MUST check cleanup operations for all Map/Set mutations. CI pipeline MUST fail if ESLint rules detect missing cleanup operations. Code review MUST block merge if Map/Set usage lacks documented key format or cleanup logic.
</enforcement>