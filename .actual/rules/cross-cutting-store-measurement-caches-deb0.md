# Adopt Array.find() and Map.get() for Collection Queries in React Components: Store Measurement Caches

These rules are ALWAYS ACTIVE for React functional components in packages/core using hooks (useCallback, useMemo, useEffect, useState) that query collections (arrays and Maps) for specific items during render cycles, event handlers, and effect callbacks.

### Rules

- **R-COLL-001** SHOULD: Store measurement caches and component references in Map structures accessed via Map.get() and Map.set().
- **R-COLL-002** SHOULD: Use Array.find() with arrow function predicates for querying arrays of configuration objects (plugins, options, zoom levels, stylesheets, path elements).
- **R-COLL-003** SHOULD: Wrap DOM collections (document.styleSheets, etc.) in Array.from() before applying Array.find() to convert collection types to arrays.
- **R-COLL-004** SHOULD: Combine Map.get() checks with early returns or default values using nullish coalescing (??) to handle missing keys gracefully.
- **R-COLL-005** MUST: Enable TypeScript strict null checks and use optional chaining (?.) or nullish coalescing (??) for all Map.get() results.
- **R-COLL-006** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in useEffect return functions.
- **R-COLL-007** MAY: Use manual for loops with break statements only when performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with large collections (>1000 items).

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Show sample Array.find() usage patterns
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Verify cache layer Map.get() implementations
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- All Map.get() results are checked with optional chaining (?.) or nullish coalescing (??)
- Map cleanup via Map.delete() is present in useEffect return functions for component-scoped caches

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and confirm the pattern adoption. TypeScript strict mode compilation MUST pass before accepting changes.
</enforcement>