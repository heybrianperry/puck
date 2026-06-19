# Use Set-Based Subscription Tracking for Reactive State Management: Use Set Data

These rules are ALWAYS ACTIVE for all React components managing virtualized drag-and-drop interactions, state subscriptions, and dynamic collections with measured heights and pinned indexes.

### Rules

- **R-SET-001** MUST: Use Set data structures (e.g., `nextPinnedIndexes.add(currentIndex)`) for tracking collections of unique identifiers where membership testing and addition are primary operations.
- **R-SET-002** MUST: Pair all `Map.set()` or `Set.add()` operations with corresponding `delete()` calls in `useEffect` cleanup functions or component removal paths.
- **R-SET-003** MUST: Ensure all `store.subscribe()` calls return unsubscribe functions that are invoked in `useEffect` cleanup phases.
- **R-SET-004** MUST: Store Map and Set references in `useRef` hooks to maintain stable references across React re-renders without triggering effects.
- **R-SET-005** SHOULD: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-SET-006** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.
- **R-SET-007** SHOULD: Implement custom serialization helpers for logging and use Map/Set-aware debugging utilities to support non-serializable structures in React DevTools.

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
- ESLint custom rules detect and flag `Set.add`/`Map.set` without corresponding `delete` in the same component.
- Unit tests assert that subscription cleanup functions are called on component unmount.

<enforcement>
Claude Code MUST NOT skip or defer verification. All Set/Map mutations require documented cleanup logic. Code review MUST block merges lacking cleanup operations or key format documentation. CI pipeline MUST fail if ESLint rules detect missing cleanup. Runtime warnings MUST be enabled in development mode for unclean subscriptions.
</enforcement>