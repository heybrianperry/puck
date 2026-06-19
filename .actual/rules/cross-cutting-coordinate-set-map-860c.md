# Use Set-Based Subscription Tracking for UI State Coordination: Coordinate Set Map

These rules are ALWAYS ACTIVE for all files in drag-and-drop and virtualized list components using @dnd-kit/react and @tanstack/react-virtual, particularly VirtualizedDropZone, DragDropContext, and components managing ephemeral UI state through Set and Map data structures.

### Rules

- **R-COORD-001** SHOULD: Coordinate Set/Map-based state updates with store subscription patterns (zoneStore.subscribe) to propagate changes across component boundaries.
- **R-COORD-002** MUST: Use Map for keyed state tracking (measurements, registrations, zone mappings) in virtualized list components to ensure O(1) lookup performance.
- **R-COORD-003** MUST: Use Set for unique collections (pinned indexes, active zones, registered drop zones) to prevent duplicate entries and enable O(1) membership testing.
- **R-COORD-004** MUST: Pair every Map.set() or Set.add() operation with a corresponding delete() call in component cleanup functions (useEffect return, componentWillUnmount) to prevent memory leaks.
- **R-COORD-005** SHOULD: Wrap Set/Map state in useRef when updates should not trigger re-renders (measuredItemHeights, rootVirtualizers pattern), but use useState when updates must trigger component updates.
- **R-COORD-006** SHOULD: Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers.
- **R-COORD-007** SHOULD: Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components.
- **R-COORD-008** MAY: Use array-based tracking with linear search only when performance profiling demonstrates that collection sizes remain guaranteed under 10 items (EXC-001).

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
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones).
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic.
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries.
- No array-based tracking with Array.includes() or Array.find() is used in performance-sensitive virtualization paths.
- Memory profiling in integration tests for drag-drop scenarios shows no leaks from missing delete() calls.

<enforcement>
Claude Code MUST NOT skip or defer verification of Set/Map cleanup patterns and store subscription coordination. Memory leaks from forgotten delete() calls are critical defects in long-running virtualized list scenarios. Code review MUST verify that every Map.set()/Set.add() has corresponding cleanup before approval.
</enforcement>