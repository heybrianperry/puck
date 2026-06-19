# Use Set-Based Subscription Tracking for Reactive State Management: Implement Subscription Patterns

These rules are ALWAYS ACTIVE for all React components and store implementations managing virtualized drag-and-drop interactions, dynamic collections with measured heights, and reactive state synchronization across multiple subscribers.

### Rules

- **R-SUB-001** MUST: Implement subscription patterns using store.subscribe() callbacks that receive state snapshots for reactive updates.
- **R-SUB-002** MUST: Store Map and Set references in useRef hooks to maintain stable references across React re-renders without triggering effects.
- **R-SUB-003** MUST: Always pair Map.set() or Set.add() operations with corresponding delete() calls in useEffect cleanup functions.
- **R-SUB-004** MUST: When implementing store.subscribe(), return an unsubscribe function and call it in the cleanup phase of useEffect.
- **R-SUB-005** MUST: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-SUB-006** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.
- **R-SUB-007** SHOULD: Implement custom serialization helpers for logging and use Map/Set-aware debugging utilities for development.

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
- All Set.add() and Map.set() operations have corresponding delete() calls in component cleanup or removal paths
- All store.subscribe() calls return unsubscribe functions that are invoked in useEffect cleanup
- Map and Set usage is documented with key format specifications in component interfaces or type definitions
- TypeScript generics are applied to all Map and Set declarations for type safety
- Compound key generation logic is centralized in shared utility functions

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if Map/Set usage lacks documented key format or cleanup logic. CI pipeline MUST fail if ESLint rules detect missing cleanup operations. Runtime warnings MUST be emitted in development mode when subscriptions are not cleaned up within expected lifecycle.
</enforcement>