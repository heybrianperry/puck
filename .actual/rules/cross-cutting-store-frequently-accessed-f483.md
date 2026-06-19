# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Store Frequently Accessed

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, style sheet references, and other in-memory data structures requiring frequent lookups by identifier.

### Rules

- **R-STORE-001** SHOULD: Store frequently accessed cached data (measurements, refs, handles) in Map structures rather than arrays when access is by key.
- **R-STORE-002** SHOULD: Use Map.get() for cache lookups in render paths and event handlers; always handle undefined return values.
- **R-STORE-003** SHOULD: Clean up Map entries in useEffect cleanup functions using Map.delete() when components unmount or items are removed.
- **R-STORE-004** SHOULD: Reserve Array.find() for configuration lookups (plugins, zoom options) and DOM collection searches where predicates are necessary.
- **R-STORE-005** MAY: Consider Map.has() before Map.get() when existence check is semantically distinct from value retrieval.
- **R-STORE-006** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders.

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
Clause Code MUST NOT skip or defer verification. Code review checklist MUST verify Map usage for caches and Array.find for searches. ESLint custom rules MUST flag Array.find() in performance-critical paths with large collections. Performance profiling in CI MUST measure render times for virtualized components. Violations require code review feedback requesting refactor or performance regression alerts trigger investigation.
</enforcement>