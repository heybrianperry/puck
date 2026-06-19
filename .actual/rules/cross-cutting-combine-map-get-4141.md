# Adopt Array.find() and Map.get() for Collection Queries in React Components: Combine Map Get

These rules are ALWAYS ACTIVE for React functional components in packages/core using hooks, cache layers implemented with Map collections, and array queries over configuration options, plugin lists, and DOM collections.

### Rules

- **R-COLL-001** MAY: Combine Map.get() with Map.set() and Map.delete() for cache lifecycle management in refs.
- **R-COLL-002** SHOULD: Use Array.find() with arrow function predicates for querying arrays of configuration objects (plugins, options, zoom levels, stylesheets, path elements).
- **R-COLL-003** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in effect cleanup functions.
- **R-COLL-004** SHOULD: Wrap DOM collection queries (document.styleSheets, etc.) in Array.from() before applying Array.find() to convert collection types to arrays.
- **R-COLL-005** SHOULD: Combine Map.get() checks with early returns or default values using nullish coalescing (??) to handle missing keys gracefully.
- **R-COLL-006** MUST NOT: Use Array.find() on collections exceeding 100 items with frequent lookups in hot paths without profiling; convert to indexed Map structures instead.
- **R-COLL-007** MUST NOT: Leave Map collections unbounded without cleanup via Map.delete(); implement cleanup in useEffect return functions.

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Verify Array.find() usage with arrow function predicates
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Confirm Map cache layer usage in refs
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

# Check for manual for loops that should be replaced with Array.find()
grep -r 'for (let.*=.*0;' packages/core/components --include='*.tsx' --include='*.ts' | grep -v 'for (let i = 0; i < .*\.length; i++)' || echo 'No suspicious manual loops found'

# Verify TypeScript strict null checks are enabled
grep -A 5 'compilerOptions' tsconfig.json | grep 'strictNullChecks'
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- Map.get() results are handled with optional chaining (?.) or nullish coalescing (??)
- useEffect cleanup functions include Map.delete() calls for cache lifecycle management

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval. Violations require documented exceptions with performance profiling data or tech lead sign-off.
</enforcement>