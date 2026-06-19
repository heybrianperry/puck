# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Use Map Set

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, cached measurements, component references, and configuration lookups that require efficient in-memory data access patterns.

### Rules

- **R-MAP-001** MUST: Use Map.set() and Map.delete() for managing lifecycle of cached entries in performance-sensitive contexts (virtualized lists, drag-drop state, DOM references, measurement data).
- **R-MAP-002** MUST: Use Map.get() for cache lookups in render paths and event handlers; always handle undefined return values with null checks or optional chaining.
- **R-MAP-003** MUST: Clean up Map entries in useEffect cleanup functions when components unmount or items are removed from cache.
- **R-MAP-004** SHOULD: Reserve Array.find() for configuration lookups (plugins, zoom options, style sheets) and DOM collection searches where predicates are necessary.
- **R-MAP-005** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders.
- **R-MAP-006** MAY: Use Map.has() before Map.get() when existence check is semantically distinct from value retrieval.

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
Clause Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes that modify data access patterns in React components.
</enforcement>