# Adopt Array.find() and Map.get() for Collection Queries in React Components: Avoid Manual Iteration

These rules are ALWAYS ACTIVE for React functional components using hooks, cache layers implemented with Map collections, array queries over configuration options, and event handlers/effect callbacks requiring item lookups in the packages/core module.

### Rules

- **R-QUERY-001** MUST_NOT: Avoid manual iteration loops (for/while) when Array.find() or Map.get() can express the query intent more clearly.
- **R-QUERY-002** MUST: Use Array.find() with arrow function predicates for single-item queries over arrays of configuration objects (plugins, options, stylesheets, path elements).
- **R-QUERY-003** MUST: Use Map.get() for keyed lookups in cache layers (measuredItemHeights, rootVirtualizers, measureRefsRef) instead of object property access.
- **R-QUERY-004** MUST: Enable TypeScript strict null checks and use optional chaining (?.) or nullish coalescing (??) for all Map.get() results to handle undefined keys gracefully.
- **R-QUERY-005** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in useEffect return functions.
- **R-QUERY-006** SHOULD: Wrap DOM collections (document.styleSheets, etc.) in Array.from() before applying Array.find() to convert collection types to arrays.
- **R-QUERY-007** MAY: Use manual for loops with break statements only when performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with collections exceeding 1000 items.

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Show sample Array.find() usage patterns
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Verify cache layer Map.get() calls
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

# Detect manual loops that should use Array.find()
grep -r 'for\s*(let\|var\|const).*in\|while\s*(' packages/core/components --include='*.tsx' --include='*.ts' | grep -v 'for.*of' | head -10
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- All Map.get() results are checked with optional chaining or nullish coalescing operators
- Cache cleanup via Map.delete() is implemented in useEffect return functions

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval. Violations require refactoring with documented rationale or approved exception with performance profiling data.
</enforcement>