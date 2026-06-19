# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Components Managing Virtualized

These rules are ALWAYS ACTIVE for all VirtualizedDropZone components, DragDropContext components, and any code managing virtualized zones using @tanstack/react-virtual and @dnd-kit/react that require cross-zone state coordination.

### Rules

- **R-VIRT-001** MUST: Components managing virtualized zones MUST subscribe to the zone store using the subscribe pattern to receive state updates for preview indexes, zone depth indexes, and area depth indexes.
- **R-VIRT-002** MUST: All zoneStore.subscribe() calls MUST have corresponding cleanup (unsubscribe or handler removal) in useEffect return functions to prevent memory leaks.
- **R-VIRT-003** MUST: All Map.set() operations for virtualizer handles and measurements MUST have corresponding Map.delete() calls in component cleanup functions.
- **R-VIRT-004** SHOULD: Subscription callbacks SHOULD be wrapped in useCallback or defined outside render to prevent unnecessary subscription churn from callback reference changes.
- **R-VIRT-005** SHOULD: Consider implementing a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change, reducing unnecessary updates.

### Verify

```bash
# Verify all virtualized zone components use zoneStore.subscribe()
grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'

# Verify Map operations for virtualizer handles and measurements
grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'

# Verify subscription cleanup in useEffect return functions
grep -r 'useEffect.*return.*(delete|unsubscribe)' packages/core/components/ --include='*.tsx'
```

**Accept when:**
- All components using virtualized zones contain zoneStore.subscribe() calls within useEffect hooks
- All Map.set() operations for virtualizer handles and measurements have corresponding Map.delete() calls in cleanup functions
- Subscription callbacks are wrapped in useCallback or defined outside render to prevent unnecessary subscription churn
- No orphaned subscriptions are detected after component unmount cycles
- Memory leak detection tests pass after mounting/unmounting virtualized zones 100 times

<enforcement>
Clause Code MUST NOT skip or defer verification. All grep verification commands MUST pass. Code review MUST verify subscription cleanup for all new virtualized components. CI pipeline MUST fail if grep verification commands detect subscribe() without cleanup patterns. Memory leak detection tests MUST pass before merge.
</enforcement>