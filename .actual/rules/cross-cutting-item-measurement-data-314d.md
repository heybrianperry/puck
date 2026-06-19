# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Item Measurement Data

These rules are ALWAYS ACTIVE for all VirtualizedDropZone and DragDropContext components using @tanstack/react-virtual and @dnd-kit/react that coordinate state changes across zone boundaries through subscription-based synchronization.

### Rules

- **R-VIRT-001** MUST: Item measurement data MUST be stored in Map structures (measuredItemHeights, measureRefsRef) keyed by component identifiers, with explicit cleanup on component unmount.
- **R-VIRT-002** MUST: All zoneStore.subscribe() calls MUST be wrapped in useEffect hooks with cleanup functions that call unsubscribe or remove handlers on component unmount.
- **R-VIRT-003** MUST: All Map.set() operations for virtualizer handles and measurements MUST have corresponding Map.delete() calls in cleanup functions.
- **R-VIRT-004** SHOULD: Subscription callbacks SHOULD be wrapped in useCallback or defined outside render to prevent unnecessary subscription churn.
- **R-VIRT-005** SHOULD: Consider implementing a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change, reducing unnecessary updates.
- **R-VIRT-006** SHOULD: Use Map.get() to check for existing entries before Map.set() operations to avoid overwriting valid data.

### Verify

```bash
# Verify all virtualized zone components use zoneStore.subscribe within useEffect
grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'

# Verify Map operations for virtualizer handles and measurements
grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'

# Verify cleanup patterns in useEffect return functions
grep -r 'useEffect.*return.*\(delete\|unsubscribe\)' packages/core/components/ --include='*.tsx'
```

**Accept when:**
- All components using virtualized zones contain zoneStore.subscribe() calls within useEffect hooks
- All Map.set() operations for virtualizer handles and measurements have corresponding Map.delete() calls in cleanup functions
- Subscription callbacks are wrapped in useCallback or defined outside render to prevent unnecessary subscription churn
- No orphaned subscriptions are detected after component unmount cycles
- Memory leak detection tests pass after mounting/unmounting virtualized zones 100 times

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep verification commands MUST pass before accepting changes to virtualized component implementations. Code review MUST verify subscription cleanup patterns for all new virtualized components. Integration tests MUST confirm no memory leaks after component mount/unmount cycles.
</enforcement>