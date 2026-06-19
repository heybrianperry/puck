# Standardize Map-Based Service Definition Lifecycle Management: Query Parameter Based

These rules are ALWAYS ACTIVE for React components and TypeScript/JavaScript files that use Map-based service registries, URLSearchParams for configuration retrieval, and React lifecycle hooks (useEffect, useCallback) for resource management.

### Rules

- **R-MAP-001** SHOULD: Query parameter-based service configuration SHOULD use URLSearchParams.get() for retrieval and validate presence before use.
- **R-MAP-002** MUST: Map-based registries MUST be wrapped in useRef to prevent recreation on every render.
- **R-MAP-003** MUST: All Map.set() operations MUST be paired with corresponding Map.delete() operations in useEffect cleanup functions.
- **R-MAP-004** SHOULD: Compound keys SHOULD be used for nested component hierarchies to prevent ID collisions (e.g., `${zoneId}:${componentId}`).
- **R-MAP-005** SHOULD: Query parameters SHOULD be extracted once at component mount and stored in state if dynamic updates are needed.

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
- All Map registries include explicit lifecycle management in useEffect cleanup functions.
- Compound keys are used consistently for nested component hierarchies to prevent collisions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint custom rules, code review checklists, and runtime leak detection in development builds are mandatory. Violations block CI pipeline and require architecture team approval for exceptions.
</enforcement>