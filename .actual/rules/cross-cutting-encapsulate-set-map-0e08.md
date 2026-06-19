# Use Set-Based Subscription Tracking for UI State Coordination: Encapsulate Set Map

These rules are ALWAYS ACTIVE for UI interaction state in drag-and-drop contexts using @dnd-kit/react, virtualized list rendering state managed by @tanstack/react-virtual, component-local caches for measurements and registrations, and store subscription callbacks that track zone depth indexes and preview states.

### Rules

- **R-SETMAP-001** SHOULD: Encapsulate Set/Map state within useRef hooks (measureRefsRef.current) when state changes should not trigger re-renders but must persist across render cycles.
- **R-SETMAP-002** MUST: Always pair Map.set() with corresponding Map.delete() in cleanup functions (useEffect return, component unmount) to prevent memory leaks.
- **R-SETMAP-003** SHOULD: Use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones) in virtualized list components.
- **R-SETMAP-004** SHOULD: Use compound keys (e.g., zoneCompound) for Map entries when tracking relationships between multiple entities such as zones and virtualizers.
- **R-SETMAP-005** SHOULD: Coordinate Set/Map updates with store subscriptions by calling store update methods after local Set/Map mutations to propagate changes to dependent components.
- **R-SETMAP-006** MAY: Use plain JavaScript objects with string keys or arrays with Array.includes() only when collection sizes remain under 10 items and simplicity outweighs performance concerns.

### Verify

```bash
# Find all Set and Map instantiations and operations
grep -r 'new Set\|new Map\|\.add(\|\.set(\|\.delete(' packages/core/components --include='*.tsx' --include='*.ts'

# Find useRef wrapping Set/Map state
grep -r 'useRef.*Map\|useRef.*Set' packages/core/components --include='*.tsx' --include='*.ts'

# Find cleanup patterns with delete operations
grep -r 'useEffect.*return.*delete\|componentWillUnmount.*delete' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- All virtualized list components use Map for keyed state (measurements, registrations) and Set for unique collections (pinned indexes, active zones)
- Every Map.set() or Set.add() operation has a corresponding delete() in component cleanup or unmount logic
- Store subscription patterns coordinate with Set/Map updates to propagate state changes across component boundaries
- Memory profiling in integration tests for drag-drop scenarios shows no leaks from missing delete() calls

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist MUST require Set/Map usage verification for virtualized list components. CI pipeline MUST scan for Set/Map patterns in components using @tanstack/react-virtual. Memory profiling MUST be run in integration tests for drag-drop scenarios to detect leaks from missing delete() calls.
</enforcement>