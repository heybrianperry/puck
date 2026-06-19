# Adopt Subscription-Based State Synchronization for Virtual Scrolling Zones: Subscription Callbacks Registered

These rules are ALWAYS ACTIVE for all VirtualizedDropZone and DragDropContext components using @tanstack/react-virtual and @dnd-kit/react that coordinate state through zoneStore subscriptions.

### Rules

- **R-VSYNC-001** MUST: Register subscription callbacks within useEffect hooks to ensure proper lifecycle management and cleanup.
- **R-VSYNC-002** MUST: Call unsubscribe or remove handlers in the useEffect cleanup function (return statement) when components unmount.
- **R-VSYNC-003** MUST: Call Map.delete() for all virtualizer handles, measurements, and measurement refs in cleanup functions corresponding to Map.set() operations.
- **R-VSYNC-004** SHOULD: Wrap subscription callbacks with useCallback to prevent subscription churn from callback reference changes on every render.
- **R-VSYNC-005** SHOULD: Implement a selector pattern (e.g., store.subscribe(selector, callback)) to only trigger callbacks when specific state slices change.
- **R-VSYNC-006** MUST: Use Map.get() to check for existing entries before Map.set() operations to avoid overwriting valid data.
- **R-VSYNC-007** MUST: Add TypeScript types for subscription callback signatures to ensure type safety across the store boundary.

### Verify

```bash
# Verify subscription callbacks are registered within useEffect hooks
grep -r 'zoneStore\.subscribe' packages/core/components/ --include='*.tsx' --include='*.ts'

# Verify Map operations for virtualizer handles and measurements
grep -r '\.set(.*,.*handle)' packages/core/components/ --include='*.tsx' | grep -E '(rootVirtualizers|measuredItemHeights|measureRefsRef)'

# Verify cleanup functions with delete/unsubscribe calls
grep -r 'useEffect.*return.*\(delete\|unsubscribe\)' packages/core/components/ --include='*.tsx'
```

**Accept when:**
- All components using virtualized zones contain zoneStore.subscribe() calls within useEffect hooks
- All Map.set() operations for virtualizer handles and measurements have corresponding Map.delete() calls in cleanup functions
- Subscription callbacks are wrapped in useCallback or defined outside render to prevent unnecessary subscription churn
- No orphaned subscriptions remain after component unmount cycles
- TypeScript types are defined for all subscription callback signatures

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST pass before accepting changes to virtualized component implementations. Memory leak detection tests MUST pass. Code review MUST verify subscription cleanup patterns for all new virtualized components.
</enforcement>