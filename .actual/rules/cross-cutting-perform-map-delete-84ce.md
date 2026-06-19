# Adopt Array.find() and Map.get() for Component State Lookup in React UI Interactions: Perform Map Delete

These rules are ALWAYS ACTIVE for React components in packages/core managing drag-drop interactions, virtualized lists, iframe style mirroring, viewport controls, and any component using useRef-stored Map instances for component ID-indexed caches.

### Rules

- **R-MAP-001** MUST: Perform Map.delete(key) cleanup operations in useEffect return functions or component unmount handlers to prevent memory leaks in long-lived component trees.
- **R-MAP-002** MUST: Store Map instances in useRef when lookup state must persist across renders without triggering re-renders: `const mapRef = useRef(new Map())`.
- **R-MAP-003** MUST: Always return cleanup functions from useEffect that call Map.delete() for component-scoped cache entries.
- **R-MAP-004** SHOULD: Wrap Array.find() results in useMemo when searching large collections or when the result is used in render or effect dependencies.
- **R-MAP-005** SHOULD: Use optional chaining or null checks when accessing Map.get() results: `const value = map.get(key)?.property`.
- **R-MAP-006** SHOULD: Convert DOM collections to arrays before using .find(): `Array.from(document.styleSheets).find(predicate)`.

### Verify

```bash
# Count Map.get() usage in packages/core/components
grep -r 'Map\.get(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Count Array.find() usage in packages/core/components
grep -r 'Array\.find(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Count Map.delete() usage in packages/core/components
grep -r 'Map\.delete(' packages/core/components --include='*.tsx' --include='*.ts' | wc -l

# Find useRef-stored Map instances
grep -r 'useRef.*new Map' packages/core/components --include='*.tsx' --include='*.ts'
```

**Accept when:**
- Map.get() and Map.delete() calls are present in at least 3 component files in packages/core/components
- Array.find() with predicate functions is used for collection searches in at least 3 component files
- useEffect return functions include Map.delete() cleanup for component-scoped cache entries in virtualized or drag-drop components

<enforcement>
Claude Code MUST NOT skip or defer verification of Map.delete() cleanup patterns in useEffect return functions and component unmount handlers. All new components using Map-based caches MUST include explicit cleanup to prevent memory leaks.
</enforcement>