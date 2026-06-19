# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Virtualizer Handles Registered

These rules are ALWAYS ACTIVE for all VirtualizedDropZone and DragDropContext components using @tanstack/react-virtual and @dnd-kit/react that coordinate state across virtualized drag-and-drop zones.

### Rules

- **R-VIRT-001** MUST: Virtualizer handles MUST be registered in the rootVirtualizers map using zone compound identifiers as keys, and MUST be removed on component unmount.
- **R-VIRT-002** MUST: All zoneStore.subscribe() calls MUST be wrapped in useEffect hooks with cleanup functions that call unsubscribe or remove handlers on component unmount.
- **R-VIRT-003** MUST: All Map.set() operations for virtualizer handles and measurements MUST have corresponding Map.delete() calls in cleanup functions.
- **R-VIRT-004** SHOULD: Subscription callbacks SHOULD be wrapped in useCallback or defined outside render to prevent unnecessary subscription churn.
- **R-VIRT-005** SHOULD: Consider implementing a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change, reducing unnecessary updates.

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
- No memory leaks are detected after mounting/unmounting virtualized zones in integration tests

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands MUST pass before code review approval. Memory leak detection tests MUST pass in CI pipeline.
</enforcement>