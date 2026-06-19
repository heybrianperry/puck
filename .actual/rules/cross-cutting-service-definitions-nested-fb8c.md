# Standardize Map-Based Service Definition Lifecycle Management: Service Definitions Nested

These rules are ALWAYS ACTIVE for React-based UI components that use Map data structures for service definition lifecycle management, including virtualizers, measurement references, and component-specific state registries.

### Rules

- **R-SVC-001** MAY: Service definitions MAY be nested within current properties of ref objects for imperative access patterns.
- **R-SVC-002** MUST: Wrap Map registries in useRef to prevent recreation on every render: `const registryRef = useRef(new Map())`.
- **R-SVC-003** MUST: Always pair set operations with delete operations in useEffect cleanup: `useEffect(() => { map.set(id, value); return () => map.delete(id); }, [id])`.
- **R-SVC-004** SHOULD: Use compound keys for nested component hierarchies to prevent ID collisions: `const key = \`${zoneId}:${componentId}\``.
- **R-SVC-005** SHOULD: Extract URLSearchParams once at component mount and store in state if dynamic updates are needed.
- **R-SVC-006** MUST: Ensure the number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions.
- **R-SVC-007** MUST: Verify all URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration.
- **R-SVC-008** MUST: Store Map-based registries in useRef hooks rather than component state or module-level variables.

### Verify

```bash
# Count Map.set() operations
grep -r 'measuredItemHeights\.set\|measureRefsRef\.current\.set\|rootVirtualizers\.set' --include='*.tsx' --include='*.ts' | wc -l

# Count Map.delete() operations
grep -r 'measuredItemHeights\.delete\|measureRefsRef\.current\.delete\|rootVirtualizers\.delete' --include='*.tsx' --include='*.ts' | wc -l

# Count URLSearchParams.get() calls
grep -r 'params\.get(' --include='*.tsx' --include='*.ts' | grep -c 'URLSearchParams\|searchParams'
```

**Accept when:**
- The number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions.
- All URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration.
- Map-based registries are stored in useRef hooks rather than component state or module-level variables.
- ESLint custom rules detect no Map operations without corresponding cleanup in useEffect dependencies.
- No Map registries exceed size thresholds indicating potential memory leaks in development builds.

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint custom rules detecting Map operations without corresponding cleanup MUST block CI pipeline. Code review rejection is mandatory for components with Map-based registries lacking explicit lifecycle management. Development console warnings MUST be generated when Map registries exceed size thresholds. Exception process requires inline LEAK-SAFE annotations and architecture team approval for deviations.
</enforcement>