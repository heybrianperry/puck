# Adopt Array.find() and Map.get() for Collection Queries in React Components: Use Array Convert

These rules are ALWAYS ACTIVE for React functional components in packages/core using hooks (useCallback, useMemo, useEffect, useState), cache layers implemented with Map collections, array queries over configuration options, and event handlers requiring item lookups.

### Rules

- **R-ARRAY-001** SHOULD: Use Array.from() to convert collection types before applying Array.find() when working with DOM APIs like document.styleSheets.
- **R-ARRAY-002** SHOULD: Use Array.find() with arrow function predicates for querying arrays of configuration objects (plugins, options, zoom levels, path elements) to retrieve single items by identity or property matching.
- **R-ARRAY-003** SHOULD: Use Map.get() for cache layer implementations (measuredItemHeights, rootVirtualizers, measureRefsRef) to access keyed values with built-in undefined semantics.
- **R-ARRAY-004** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in useEffect return functions.
- **R-ARRAY-005** SHOULD: Combine Map.get() checks with early returns or default values using nullish coalescing (??) or optional chaining (?.) to handle missing keys gracefully.
- **R-ARRAY-006** MAY: Use manual for loops with break statements only when performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with collections exceeding 100 items with frequent lookups.

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Verify Array.find() usage patterns with arrow function predicates
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Check Map cache layer implementations
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

# Verify Array.from() usage with DOM collections
grep -r 'Array.from.*styleSheets\|Array.from.*document\.' packages/core/components --include='*.tsx'

# Ensure no manual for loops with single-item returns
grep -r 'for.*{.*return' packages/core/components --include='*.tsx' | grep -v '//' | wc -l
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- Array.from() is used before Array.find() when querying DOM collections like document.styleSheets
- Map.get() results are handled with optional chaining (?.) or nullish coalescing (??) operators

<enforcement>
Clause Code MUST NOT skip or defer verification. All R-ARRAY rules MUST be checked during code review. TypeScript strict mode compilation is mandatory. ESLint custom rules detecting manual iteration patterns MUST pass before PR merge.
</enforcement>