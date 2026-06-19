# Adopt Array.find() and Map.get() for Collection Queries in React Components: Use Map Get

These rules are ALWAYS ACTIVE for React functional components using hooks in the packages/core module, particularly those implementing UI interactions with collections stored in refs and state (VirtualizedDropZone, DragDropContext, AutoFrame, ViewportControls, Layout, and similar components).

### Rules

- **R-MAPGET-001** MUST: Use Map.get() for keyed lookups in Map collections rather than iterating over entries.
- **R-MAPGET-002** MUST: Use Array.find() with arrow function predicates for single-item queries over arrays of configuration objects (plugins, options, stylesheets, path elements).
- **R-MAPGET-003** MUST: Wrap DOM collection queries (document.styleSheets, etc.) in Array.from() before applying Array.find().
- **R-MAPGET-004** MUST: Enable TypeScript strict null checks and use optional chaining (?.) or nullish coalescing (??) for all Map.get() results.
- **R-MAPGET-005** SHOULD: Store component-scoped caches in useRef with Map instances, accessed via Map.get() and cleaned up via Map.delete() in useEffect return functions.
- **R-MAPGET-006** SHOULD: Combine Map.get() checks with early returns or default values using nullish coalescing to handle missing keys gracefully.
- **R-MAPGET-007** MAY: Use manual for loops with break statements only when performance profiling demonstrates Array.find() creates measurable bottlenecks in hot paths with large collections (>1000 items).

### Verify

```bash
# Confirm Array.find() and Map.get() usage across component files
grep -r 'Array.find\|Map.get' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Identify Array.find() usage patterns
grep -r '\.find((.*) =>' packages/core/components --include='*.tsx' | head -5

# Verify Map.get() usage in cache layers
grep -r 'measuredItemHeights.get\|rootVirtualizers.get\|measureRefsRef.current.get' packages/core/components --include='*.tsx'

# Check for manual for loops that should be refactored
grep -r 'for (let.*=.*0;' packages/core/components --include='*.tsx' --include='*.ts' | grep -v 'node_modules'

# Verify TypeScript strict mode compilation
tsc --strict --noEmit
```

**Accept when:**
- Grep commands confirm Array.find() and Map.get() usage in at least 5 component files under packages/core/components
- Code review identifies no manual for loops that could be replaced with Array.find() for single-item queries
- TypeScript compilation succeeds with strict null checks enabled for all Map.get() call sites
- All Map.get() results are checked with optional chaining or nullish coalescing operators
- Cache layers (measuredItemHeights, rootVirtualizers, measureRefsRef) use Map.get() consistently
- Array.from() is applied to DOM collections before Array.find() is called

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-MAPGET rules must be verified through code review and TypeScript compilation before accepting changes to React components in packages/core. ESLint custom rules and CI pipeline checks must enforce these patterns.
</enforcement>