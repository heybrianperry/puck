# Use Set-Based Subscription Tracking for UI State Coordination: Clean Map Entries

These rules are ALWAYS ACTIVE for all UI interaction state in drag-and-drop contexts using @dnd-kit/react, virtualized list rendering state managed by @tanstack/react-virtual, component-local caches for measurements and registrations, and store subscription callbacks that track zone depth indexes and preview states.

### Rules

- **R-SETMAP-001** MUST: Clean up Map entries using delete() when components unmount or items are removed to prevent memory leaks in long-lived virtualized lists.
- **R-SETMAP-002** MUST: Pair Map.set() or Set.add() operations with corresponding delete() calls in cleanup functions (useEffect return, component unmount).
- **R-SETMAP-003** SHOULD: Use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones) in virtualized list components.
- **R-SETMAP-004** SHOULD: Wrap Set/Map state in useRef when updates should not trigger re-renders, but use useState when updates must trigger component updates.
- **R-SETMAP-005** SHOULD: Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers.
- **R-SETMAP-006** MAY: Add temporary console.log statements in store subscriptions to inspect Set/Map state changes during development.

### Verify

```bash
# Find all Set and Map instantiations and operations
grep -r 'new Set\|new Map\|\.add(\|\.set(\|\.delete(' packages/core/components --include='*.tsx' --include='*.ts'

# Find useRef patterns with Set/Map
grep -r 'useRef.*Map\|useRef.*Set' packages/core/components --include='*.tsx' --include='*.ts'

# Find cleanup patterns with delete operations
grep -r 'useEffect.*return.*delete\|componentWillUnmount.*delete' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones).
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic.
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries.
- Memory profiling in integration tests for drag-drop scenarios detects no leaks from missing delete() calls.

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist MUST require Set/Map usage verification for virtualized list components. Automated grep-based checks in CI pipeline MUST scan for Set/Map patterns in components using @tanstack/react-virtual. Memory profiling in integration tests MUST detect leaks from missing delete() calls. Performance regression alerts MUST trigger if virtualized list scroll performance degrades below baseline thresholds.
</enforcement>