# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: State Updates That

These rules are ALWAYS ACTIVE for all VirtualizedDropZone components, DragDropContext components, and state synchronization code in virtualized drag-and-drop systems using @tanstack/react-virtual and @dnd-kit/react.

### Rules

- **R-VSYNC-001** MUST: State updates that affect multiple zones MUST use Set data structures (nextPinnedIndexes) to track unique indexes and avoid duplicate processing.
- **R-VSYNC-002** MUST: All zoneStore.subscribe() calls MUST be wrapped in useEffect hooks with cleanup functions that call unsubscribe or remove handlers on component unmount.
- **R-VSYNC-003** MUST: All Map.set() operations for virtualizer handles and measurements MUST have corresponding Map.delete() calls in cleanup functions.
- **R-VSYNC-004** SHOULD: Subscription callbacks SHOULD be wrapped in useCallback or defined outside render to prevent unnecessary subscription churn.
- **R-VSYNC-005** SHOULD: Map.get() SHOULD be used to check for existing entries before Map.set() operations to avoid overwriting valid data.
- **R-VSYNC-006** MAY: Consider implementing a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change, reducing unnecessary updates.

### Verify

```bash
# Verify all virtualized zone components use zoneStore.subscribe() within useEffect hooks
grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'

# Verify Map operations for virtualizer handles and measurements
grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'

# Verify subscription cleanup patterns in useEffect return functions
grep -r 'useEffect.*return.*\(delete\|unsubscribe\)' packages/core/components/ --include='*.tsx'
```

**Accept when:**
- All components using virtualized zones contain zoneStore.subscribe() calls within useEffect hooks
- All Map.set() operations for virtualizer handles and measurements have corresponding Map.delete() calls in cleanup functions
- Subscription callbacks are wrapped in useCallback or defined outside render to prevent unnecessary subscription churn
- No orphaned subscriptions are detected after component unmount cycles
- TypeScript types are defined for subscription callback signatures

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and return expected patterns. Memory leak detection tests MUST pass. Code review MUST verify subscription cleanup before merge.
</enforcement>