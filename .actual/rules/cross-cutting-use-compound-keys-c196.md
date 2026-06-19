# Use Set-Based Subscription Tracking for UI State Coordination: Use Compound Keys

These rules are ALWAYS ACTIVE for all UI interaction state coordination in drag-and-drop contexts using @dnd-kit/react, virtualized list rendering managed by @tanstack/react-virtual, component-local caches for measurements and registrations, and store subscription callbacks that track zone depth indexes and preview states.

### Rules

- **R-COMPOUND-001** MAY: Use compound keys (zoneCompound) for Map entries when tracking state across multiple dimensions such as zone and virtualizer pairings.
- **R-COMPOUND-002** MUST: Wrap Set/Map state in useRef when updates should not trigger re-renders (measureRefsRef.current pattern), but use useState when updates must trigger component updates.
- **R-COMPOUND-003** MUST: Always pair Map.set() with corresponding Map.delete() in cleanup functions (useEffect return, component unmount) to prevent memory leaks.
- **R-COMPOUND-004** SHOULD: Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components.
- **R-COMPOUND-005** SHOULD: Add temporary console.log statements in store subscriptions to inspect Set/Map state changes during development for debugging purposes.

### Verify

```bash
# Detect Set and Map usage patterns in virtualized components
grep -r 'new Set\|new Map\|\.add(\|\.set(\|\.delete(' packages/core/components --include='*.tsx' --include='*.ts'

# Verify useRef wrapping of Set/Map state
grep -r 'useRef.*Map\|useRef.*Set' packages/core/components --include='*.tsx' --include='*.ts'

# Verify cleanup patterns with delete operations
grep -r 'useEffect.*return.*delete\|componentWillUnmount.*delete' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones)
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries
- Compound keys are used for Map entries tracking relationships between multiple entities such as zones and virtualizers

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist MUST require Set/Map usage verification for virtualized list components. Automated grep-based checks in CI pipeline MUST scan for Set/Map patterns in components using @tanstack/react-virtual. Memory profiling in integration tests MUST detect leaks from missing delete() calls. Performance regression alerts MUST trigger if virtualized list scroll performance degrades below baseline thresholds.
</enforcement>