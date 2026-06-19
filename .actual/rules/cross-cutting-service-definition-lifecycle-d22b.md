# Standardize Map-Based Service Definition Lifecycle Management: Service Definition Lifecycle

These rules are ALWAYS ACTIVE for React components and TypeScript/JavaScript files that use Map-based service registries, virtualizers, measurement references, or URLSearchParams-based configuration retrieval in client-side rendering contexts.

### Rules

- **R-SDL-001** MUST: Service definition lifecycle MUST include explicit get, set, and delete operations coordinated with component mount/unmount cycles.
- **R-SDL-002** MUST: Map registries MUST be wrapped in useRef to prevent recreation on every render.
- **R-SDL-003** MUST: All Map.set() operations MUST be paired with corresponding Map.delete() operations in useEffect cleanup functions.
- **R-SDL-004** MUST: Compound keys (e.g., `${zoneId}:${componentId}`) MUST be used for nested component hierarchies to prevent ID collisions.
- **R-SDL-005** SHOULD: Query parameter configuration SHOULD be extracted once at component mount and stored in state if dynamic updates are needed.

### Verify

```bash
# Count Map.set() operations across service registries
grep -r 'measuredItemHeights\.set\|measureRefsRef\.current\.set\|rootVirtualizers\.set' --include='*.tsx' --include='*.ts' | wc -l

# Count corresponding Map.delete() operations
grep -r 'measuredItemHeights\.delete\|measureRefsRef\.current\.delete\|rootVirtualizers\.delete' --include='*.tsx' --include='*.ts' | wc -l

# Verify URLSearchParams usage patterns
grep -r 'params\.get(' --include='*.tsx' --include='*.ts' | grep -c 'URLSearchParams\|searchParams'
```

**Accept when:**
- The number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions.
- All URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration.
- Map-based registries are stored in useRef hooks rather than component state or module-level variables.
- All Map registries use compound keys that include scope identifiers to prevent collisions.
- No Map.set() operations exist without paired delete operations in useEffect cleanup blocks.

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint custom rules, code review checklists, and runtime leak detection in development builds are mandatory enforcement mechanisms. Violations block CI pipeline and require architecture team approval for exceptions.
</enforcement>