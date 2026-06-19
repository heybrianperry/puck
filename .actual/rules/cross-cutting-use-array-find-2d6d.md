# Adopt Array.find() and Map.get() for In-Memory Data Access in React Components: Use Array Find

These rules are ALWAYS ACTIVE for React components in the core package managing virtualized lists, drag-drop state, cached measurements, component references, and configuration lookups.

### Rules

- **R-ARRAY-FIND-001** SHOULD: Use Array.find() with predicate functions when searching collections for items matching specific criteria, particularly for configuration lookups (zoom options, plugins, style sheets) and DOM element searches.
- **R-ARRAY-FIND-002** SHOULD: Use Map.get() for O(1) cache lookups of keyed data such as measured item heights, virtualizer handles, and component references in performance-sensitive rendering paths.
- **R-ARRAY-FIND-003** MUST: Clean up Map entries in useEffect cleanup functions using Map.delete() when components unmount or items are removed to prevent memory leaks.
- **R-ARRAY-FIND-004** SHOULD: Store component measurement caches and ref maps in useRef to persist across renders without triggering re-renders.
- **R-ARRAY-FIND-005** SHOULD: Always handle undefined return values from Map.get() operations using optional chaining or explicit null checks.
- **R-ARRAY-FIND-006** MAY: Consider Map.has() before Map.get() when existence check is semantically distinct from value retrieval.

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
Claude Code MUST NOT skip or defer verification. All three verification conditions must pass before accepting this rule as active in the codebase.
</enforcement>