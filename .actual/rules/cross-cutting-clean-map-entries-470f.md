# Use Set-Based Subscription Tracking for Reactive State Management: Clean Map Entries

These rules are ALWAYS ACTIVE for all React components using Set and Map data structures for subscription tracking, virtualization state management, and drag-and-drop interactions.

### Rules

- **R-SUBS-001** MUST: Clean up Map entries using delete() operations when components unmount or items are removed to prevent memory leaks.
- **R-SUBS-002** MUST: Pair all Map.set() and Set.add() operations with corresponding delete() calls in useEffect cleanup functions.
- **R-SUBS-003** MUST: Ensure store.subscribe() calls return unsubscribe functions that are invoked in useEffect cleanup phases.
- **R-SUBS-004** MUST: Store Map and Set references in useRef hooks to maintain stable references across React re-renders without triggering effects.
- **R-SUBS-005** SHOULD: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-SUBS-006** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.

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
- useRef hooks are used to store Map and Set references to prevent unnecessary re-renders
- Compound key generation logic is centralized in shared utility functions

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if Map/Set usage lacks documented key format or cleanup logic. CI pipeline MUST fail if ESLint rules detect missing cleanup operations. Runtime warnings MUST be emitted in development mode when subscriptions are not cleaned up within expected lifecycle.
</enforcement>