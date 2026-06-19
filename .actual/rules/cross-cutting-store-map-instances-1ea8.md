# Adopt Array.find() and Map.get() for Component State Lookup in React UI Interactions: Store Map Instances

These rules are ALWAYS ACTIVE for React components in packages/core managing drag-drop interactions, virtualized lists, iframe style mirroring, viewport controls, and any component using useRef-stored Map instances for component ID-indexed caches.

### Rules

- **R-MAP-001** SHOULD: Store Map instances in useRef containers (e.g., measureRefsRef.current) when lookup state must persist across renders without triggering re-renders.
- **R-MAP-002** MUST: Always return cleanup functions from useEffect that call Map.delete() for component-scoped cache entries to prevent memory leaks.
- **R-MAP-003** SHOULD: Use Map.get() for O(1) lookup performance when retrieving component measurements and references indexed by string identifiers (componentId, zoneCompound).
- **R-MAP-004** SHOULD: Use Array.find() with explicit predicates for type-safe, predicate-based search in collections where identity depends on property comparison.
- **R-MAP-005** SHOULD: Convert DOM collections to arrays before using .find(): Array.from(document.styleSheets).find(predicate).
- **R-MAP-006** SHOULD: Wrap Array.find() results in useMemo when searching large collections or when the result is used in render or effect dependencies.
- **R-MAP-007** SHOULD: Use optional chaining or null checks when accessing Map.get() results: const value = map.get(key)?.property.

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
Claude Code MUST NOT skip or defer verification of these rules when reviewing or modifying React components in packages/core that manage component state lookup, virtualized rendering, drag-drop interactions, or iframe style mirroring.
</enforcement>