# Use Set-Based Subscription Tracking for Reactive State Management: Use Compound Keys

These rules are ALWAYS ACTIVE for all React components using @dnd-kit drag-and-drop interactions, virtualized collections, and custom store implementations with subscription patterns.

### Rules

- **R-COMPOUND-001** SHOULD: Use compound keys (e.g., zoneCompound) when indexing virtualizers or handlers that span multiple logical boundaries.
- **R-COMPOUND-002** MUST: Pair all Map.set() or Set.add() operations with corresponding delete() calls in useEffect cleanup functions.
- **R-COMPOUND-003** MUST: Ensure all store.subscribe() calls return unsubscribe functions that are invoked in useEffect cleanup phases.
- **R-COMPOUND-004** SHOULD: Store Map and Set references in useRef hooks to maintain stable references across React re-renders without triggering effects.
- **R-COMPOUND-005** SHOULD: Use TypeScript generics to type Map keys and values, ensuring compile-time safety for compound key structures.
- **R-COMPOUND-006** SHOULD: Centralize compound key generation in shared utility functions and document key format in component interfaces.

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
- All Set.add() and Map.set() operations have corresponding delete() calls in component cleanup or removal paths.
- All store.subscribe() calls return unsubscribe functions that are invoked in useEffect cleanup.
- Map and Set usage is documented with key format specifications in component interfaces or type definitions.
- No orphaned Map entries or failed lookups occur due to inconsistent compound key generation.
- Subscription cleanup functions are verified to be called on component unmount in unit tests.

<enforcement>
Clause Code MUST NOT skip or defer verification. All violations detected by ESLint rules or code review must block merge. Runtime warnings in development mode must be addressed before production deployment. Exceptions require tech lead approval and inline documentation linking to this architectural decision.
</enforcement>