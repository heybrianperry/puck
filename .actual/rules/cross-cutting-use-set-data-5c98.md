# Use Set-Based Subscription Tracking for UI State Coordination: Use Set Data

These rules are ALWAYS ACTIVE for all UI interaction state coordination in drag-and-drop contexts using @dnd-kit/react, virtualized list rendering managed by @tanstack/react-virtual, component-local caches for measurements and registrations, and store subscription callbacks that track zone depth indexes and preview states.

### Rules

- **R-SET-001** MUST: Use Set data structures for tracking unique UI state identifiers such as pinned indexes, active zones, or selected items where membership testing is the primary operation.
- **R-SET-002** MUST: Pair every Map.set() or Set.add() operation with a corresponding delete() in component cleanup functions or unmount logic to prevent memory leaks.
- **R-SET-003** SHOULD: Wrap Set/Map state in useRef when updates should not trigger re-renders, but use useState when updates must trigger component updates.
- **R-SET-004** SHOULD: Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers.
- **R-SET-005** SHOULD: Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components.
- **R-SET-006** MAY: Use array-based tracking with linear search only when performance profiling demonstrates that approach is faster for collections guaranteed to remain under 10 items (EXC-001).

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
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones)
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries
- No array-based membership testing (Array.includes(), Array.find()) is used in virtualized list measurement or zone registration paths
- Memory profiling in integration tests for drag-drop scenarios shows no leaks from missing delete() calls

<enforcement>
Claude Code MUST NOT skip or defer verification of Set/Map usage patterns, cleanup logic, and store subscription coordination. All three verify commands MUST execute successfully before accepting changes to virtualized list or drag-drop components.
</enforcement>