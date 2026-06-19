# Adopt Array.find() and Map.get() for Component State Lookup in React UI Interactions: Use Array Convert

These rules are ALWAYS ACTIVE for React components in packages/core managing drag-drop interactions, virtualized lists, iframe style mirroring, viewport controls, and any component using useRef-stored Map instances for component ID-indexed caches.

### Rules

- **R-ARRAY-CONVERT-001** SHOULD: Use Array.from() to convert DOM collections (e.g., document.styleSheets) to arrays before applying .find() to ensure consistent iteration semantics.
- **R-ARRAY-CONVERT-002** SHOULD: Use Map.get() for O(1) lookup performance when retrieving component measurements and references indexed by string identifiers (componentId, zoneCompound).
- **R-ARRAY-CONVERT-003** SHOULD: Use Array.find() with explicit predicates for type-safe, predicate-based search in collections where identity depends on property comparison (path matching, href matching, value matching).
- **R-ARRAY-CONVERT-004** MUST: Include Map.delete() cleanup in useEffect return functions to prevent memory leaks in components with dynamic lifecycles.
- **R-ARRAY-CONVERT-005** SHOULD: Store Map instances in useRef when lookup state must persist across renders without triggering re-renders.
- **R-ARRAY-CONVERT-006** SHOULD: Wrap Array.find() results in useMemo when searching large collections or when the result is used in render or effect dependencies.
- **R-ARRAY-CONVERT-007** SHOULD: Use optional chaining or null checks when accessing Map.get() results to handle undefined values for missing keys.

### Verify

```bash
# Count Map.get() usage in packages/core/components
grep -r 'Map\.get(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Count Array.find() usage in packages/core/components
grep -r 'Array\.find(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Count Map.delete() usage in packages/core/components
grep -r 'Map\.delete(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Find useRef-wrapped Map instances
grep -r 'useRef.*new Map' packages/core/components --include='*.tsx' --include='*.ts'

# Verify Array.from() usage with DOM collections
grep -r 'Array\.from(document\.' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Map.get() and Map.delete() calls are present in at least 3 component files in packages/core/components
- Array.find() with predicate functions is used for collection searches in at least 3 component files
- useEffect return functions include Map.delete() cleanup for component-scoped cache entries in virtualized or drag-drop components
- Array.from() is used to convert DOM collections before applying .find() in at least 2 component files

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations block merges and require documented exceptions with frontend architecture team approval.
</enforcement>