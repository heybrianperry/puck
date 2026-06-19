# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Subscriptions Include Console

These rules are ALWAYS ACTIVE for all VirtualizedDropZone and DragDropContext components using @tanstack/react-virtual and @dnd-kit/react that coordinate state through zoneStore subscriptions.

### Rules

- **R-VSYNC-001** MUST: All components using virtualized zones contain `zoneStore.subscribe()` calls within `useEffect` hooks with corresponding cleanup functions that call unsubscribe or remove handlers on component unmount.
- **R-VSYNC-002** MUST: All `Map.set()` operations for virtualizer handles and measurements have corresponding `Map.delete()` calls in cleanup functions to prevent memory leaks.
- **R-VSYNC-003** MUST: Subscription callbacks are wrapped in `useCallback` or defined outside render to prevent unnecessary subscription churn from callback reference changes.
- **R-VSYNC-004** MAY: Subscriptions MAY include console logging for debugging preview index and depth index changes during development.
- **R-VSYNC-005** SHOULD: Implement a selector pattern (e.g., `store.subscribe(selector, callback)`) to only trigger callbacks when specific state slices change, reducing unnecessary updates.
- **R-VSYNC-006** SHOULD: Use `Map.get()` to check for existing entries before `Map.set()` operations to avoid overwriting valid data.

### Verify

```bash
# Verify zoneStore.subscribe() usage in virtualized components
grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'

# Verify Map operations for virtualizer handles and measurements
grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'

# Verify cleanup patterns in useEffect
grep -r 'useEffect.*return.*\(delete\|unsubscribe\)' packages/core/components/ --include='*.tsx'
```

**Accept when:**
- All components using virtualized zones contain `zoneStore.subscribe()` calls within `useEffect` hooks
- All `Map.set()` operations for virtualizer handles and measurements have corresponding `Map.delete()` calls in cleanup functions
- Subscription callbacks are wrapped in `useCallback` or defined outside render to prevent unnecessary subscription churn
- No orphaned subscriptions are detected after mounting/unmounting virtualized zones
- Console logging for debugging is present but does not impact production performance

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and return expected patterns. Code review MUST verify subscription cleanup before merge. Integration tests MUST confirm no memory leaks after 100 mount/unmount cycles.
</enforcement>