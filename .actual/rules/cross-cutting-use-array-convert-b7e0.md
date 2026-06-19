# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Use Array Convert

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, cached measurements, component references, and style sheet lookups.

### Rules

- **R-ARRAY-CONVERT-001** MUST: Use Array.from() to convert DOM collections (e.g., document.styleSheets) to arrays before applying find() operations
- **R-ARRAY-CONVERT-002** MUST: Use Map.get() for cache lookups in render paths and event handlers; always handle undefined return values
- **R-ARRAY-CONVERT-003** MUST: Clean up Map entries in useEffect cleanup functions (e.g., measureRefsRef.current.delete(componentId))
- **R-ARRAY-CONVERT-004** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders
- **R-ARRAY-CONVERT-005** SHOULD: Reserve Array.find() for configuration lookups (plugins, zoom options) and DOM collection searches where predicates are necessary
- **R-ARRAY-CONVERT-006** SHOULD: Consider Map.has() before Map.get() when existence check is semantically distinct from value retrieval

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
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting code that modifies in-memory data access patterns in React components.
</enforcement>