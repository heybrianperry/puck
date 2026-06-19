# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Combine Map Array

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, cached measurements, component references, and configuration lookups.

### Rules

- **R-MAPARRAY-001** MUST: Use Map.get() for O(1) keyed access to cached measurements, component references, and virtualizer handles in performance-critical rendering paths.
- **R-MAPARRAY-002** MUST: Clean up Map entries in useEffect cleanup functions using Map.delete() when components unmount or items are removed.
- **R-MAPARRAY-003** MUST: Handle undefined return values from Map.get() with null checks or optional chaining.
- **R-MAPARRAY-004** SHOULD: Use Array.find() for predicate-based searches in configuration lookups (zoom options, plugins, style sheets) and DOM collection searches.
- **R-MAPARRAY-005** MAY: Combine Map and Array data structures when both keyed access and iteration are required.
- **R-MAPARRAY-006** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders.
- **R-MAPARRAY-007** SHOULD NOT: Use Array.find() in hot paths (useCallback, useMemo) with large collections; profile and refactor to Map-based access if performance degrades.

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
Clause Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes that introduce new data access patterns in React components.
</enforcement>