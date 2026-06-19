# Use Set-Based Subscription Tracking for UI State Coordination: Use Map Data

These rules are ALWAYS ACTIVE for all UI interaction state coordination in drag-and-drop contexts using @dnd-kit/react, virtualized list rendering managed by @tanstack/react-virtual, component-local caches for measurements and registrations, and store subscription callbacks that track zone depth indexes and preview states.

### Rules

- **R-MAP-001** MUST: Use Map data structures for keyed UI state such as measured item heights, virtualizer handles, or component-specific measurement references where O(1) get/set/delete operations are required.
- **R-MAP-002** MUST: Pair every Map.set() or Set.add() operation with a corresponding delete() in component cleanup functions (useEffect return, component unmount) to prevent memory leaks.
- **R-MAP-003** SHOULD: Wrap Set/Map state in useRef when updates should not trigger re-renders (measureRefsRef.current pattern), but use useState when updates must trigger component updates.
- **R-MAP-004** SHOULD: Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers.
- **R-MAP-005** SHOULD: Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components.
- **R-MAP-006** MAY: Use array-based tracking with linear search only when performance profiling demonstrates that approach is faster for collections guaranteed to remain under 10 items (EXC-001).

### Verify

```bash
# Check for Set and Map usage patterns in virtualized components
grep -r 'new Set\|new Map\|\.add(\|\.set(\|\.delete(' packages/core/components --include='*.tsx' --include='*.ts'

# Check for useRef wrapping of Set/Map state
grep -r 'useRef.*Map\|useRef.*Set' packages/core/components --include='*.tsx' --include='*.ts'

# Check for cleanup patterns with delete operations
grep -r 'useEffect.*return.*delete\|componentWillUnmount.*delete' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones).
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic.
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries.
- Memory profiling in integration tests for drag-drop scenarios detects no leaks from missing delete() calls.
- Virtualized list scroll performance meets or exceeds baseline thresholds.

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist MUST require Set/Map usage verification for virtualized list components. CI pipeline MUST run automated grep-based checks scanning for Set/Map patterns in components using @tanstack/react-virtual. Memory profiling MUST be performed in integration tests for drag-drop scenarios to detect leaks from missing delete() calls. Performance regression alerts MUST trigger if virtualized list scroll performance degrades below baseline thresholds.
</enforcement>