# Adopt Array.find() and Map.get() for Collection Queries in React Components: Use Array Find

These rules are ALWAYS ACTIVE for React functional components using hooks in the packages/core module, particularly those implementing UI interactions with collections stored in refs and state (VirtualizedDropZone, DragDropContext, AutoFrame, ViewportControls, Layout, and similar components).

### Rules

- **R-ARRAY-FIND-001** MUST: Use Array.find() with predicate functions for querying arrays when a single matching element is required, rather than manual for loops or filter operations that return single items.

- **R-ARRAY-FIND-002** MUST: Use Map.get() for keyed lookups in cache layers (measuredItemHeights, rootVirtualizers, measureRefsRef) instead of object property access or manual iteration.

- **R-ARRAY-FIND-003** MUST: Enable TypeScript strict null checks and handle undefined returns from Map.get() explicitly using optional chaining (?.) or nullish coalescing (??) operators.

- **R-ARRAY-FIND-004** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in useEffect return functions.

- **R-ARRAY-FIND-005** SHOULD: Wrap DOM collections (document.styleSheets, etc.) with Array.from() before applying Array.find() to convert collection types to arrays.

- **R-ARRAY-FIND-006** MAY: Use manual for loops with break statements only when performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with collections exceeding 1000 items; document with profiling data and ticket reference.

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Show sample Array.find() usage patterns
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Verify cache layer Map.get() calls
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

# Check for manual loops that should use Array.find()
grep -r 'for (let.*=.*0;' packages/core/components --include='*.tsx' --include='*.ts' | grep -v 'for (let i = 0; i < .*\.length; i++)' | head -10
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- All Map.get() results are handled with optional chaining or nullish coalescing operators
- Cache cleanup via Map.delete() is present in useEffect return functions where applicable

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules R-ARRAY-FIND-001 through R-ARRAY-FIND-006 must be validated before accepting changes to collection query patterns in React components.
</enforcement>