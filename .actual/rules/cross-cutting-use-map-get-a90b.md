# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Use Map Get

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, cached measurements, component references, and style sheet lookups.

### Rules

- **R-MAP-001** MUST: Use Map.get() for O(1) lookups when accessing cached data by identifier (e.g., component IDs, zone compounds, measured item heights, virtualizer handles).
- **R-MAP-002** MUST: Clean up Map entries in useEffect cleanup functions using Map.delete() when components unmount or items are removed.
- **R-MAP-003** MUST: Handle undefined return values from Map.get() with null checks or optional chaining.
- **R-MAP-004** SHOULD: Use Array.find() for predicate-based searches in configuration lookups (plugins, zoom options, style sheets) and DOM collection searches.
- **R-MAP-005** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders.
- **R-MAP-006** MAY: Consider Map.has() before Map.get() when existence check is semantically distinct from value retrieval.

### Verify

```bash
# Count Map.get() usage across core components
grep -r 'Map.*\.get(' packages/core/components/ | wc -l

# Count Array.find() usage across core components
grep -r '\.find(' packages/core/components/ | wc -l

# Verify Map.delete() calls appear in useEffect cleanup contexts
grep -r 'Map.*\.delete(' packages/core/components/ | grep -c 'useEffect\|cleanup'
```

**Accept when:**
- Map.get() usage count is greater than 10 across core components, indicating established pattern
- Array.find() usage is present in at least 3 component files for configuration/collection searches
- Map.delete() calls appear in useEffect cleanup contexts, demonstrating proper lifecycle management

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verification conditions must pass before accepting this rule as active in the codebase.
</enforcement>