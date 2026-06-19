# Adopt Array.find() and Map.get() for Component State Lookup in React UI Interactions: Use Array Find

These rules are ALWAYS ACTIVE for React components in packages/core managing drag-drop interactions, virtualized lists, iframe style mirroring, viewport controls, and any component using useRef-stored Map instances for component ID-indexed caches.

### Rules

- **R-ARRAY-FIND-001** MUST: Use Array.find(predicate) for locating elements in collections where identity is determined by property comparison (e.g., path matching, href matching, option value matching).
- **R-ARRAY-FIND-002** MUST: Store Map instances in useRef when lookup state must persist across renders without triggering re-renders: `const mapRef = useRef(new Map())`.
- **R-ARRAY-FIND-003** MUST: Always return cleanup functions from useEffect that call Map.delete() for component-scoped cache entries to prevent memory leaks.
- **R-ARRAY-FIND-004** MUST: Convert DOM collections to arrays before using .find(): `Array.from(document.styleSheets).find(predicate)`.
- **R-ARRAY-FIND-005** SHOULD: Wrap Array.find() results in useMemo when searching large collections or when the result is used in render or effect dependencies.
- **R-ARRAY-FIND-006** SHOULD: Use optional chaining or null checks when accessing Map.get() results: `const value = map.get(key)?.property`.

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
```

**Accept when:**
- Map.get() and Map.delete() calls are present in at least 3 component files in packages/core/components
- Array.find() with predicate functions is used for collection searches in at least 3 component files
- useEffect return functions include Map.delete() cleanup for component-scoped cache entries in virtualized or drag-drop components

<enforcement>
Claude Code MUST NOT skip or defer verification. All new React UI interaction components in packages/core MUST conform to these rules. Code review MUST verify Map.get()/Array.find() pattern usage. CI pipeline MUST fail if components use object bracket notation for component ID-indexed caches without documented justification. Memory profiling tests MUST flag components with growing Map instances lacking proper cleanup.
</enforcement>