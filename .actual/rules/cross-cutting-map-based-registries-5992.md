# Standardize Map-Based Service Definition Lifecycle Management: Map Based Registries

These rules are ALWAYS ACTIVE for React components using Map-based service registries, virtualizers, measurement references, and lifecycle-managed resources in client-side rendering contexts.

### Rules

- **R-MAP-001** SHOULD: Map-based registries SHOULD use ref objects (useRef) to maintain stable references across render cycles.
- **R-MAP-002** MUST: Always pair Map.set() operations with corresponding Map.delete() operations in useEffect cleanup functions.
- **R-MAP-003** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) for nested component hierarchies to prevent ID collisions.
- **R-MAP-004** SHOULD: Extract URLSearchParams query parameters once at component mount and store in state if dynamic updates are needed.
- **R-MAP-005** MUST: Wrap Map registries in useRef to prevent recreation on every render: `const registryRef = useRef(new Map())`.

### Verify

```bash
# Count Map.set() operations across the codebase
grep -r 'measuredItemHeights\.set\|measureRefsRef\.current\.set\|rootVirtualizers\.set' --include='*.tsx' --include='*.ts' | wc -l

# Count Map.delete() operations across the codebase
grep -r 'measuredItemHeights\.delete\|measureRefsRef\.current\.delete\|rootVirtualizers\.delete' --include='*.tsx' --include='*.ts' | wc -l

# Count URLSearchParams.get() calls with proper instantiation
grep -r 'params\.get(' --include='*.tsx' --include='*.ts' | grep -c 'URLSearchParams\|searchParams'
```

**Accept when:**
- The number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions.
- All URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration.
- Map-based registries are stored in useRef hooks rather than component state or module-level variables.
- All Map registries follow the pattern: `const registryRef = useRef(new Map())`.
- Compound keys are used consistently for nested component hierarchies to prevent collisions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint custom rules detecting Map operations without corresponding cleanup MUST block CI pipeline. Code review rejection is mandatory for components with Map-based registries lacking explicit lifecycle management. Development console warnings MUST appear when Map registries exceed size thresholds indicating potential leaks.
</enforcement>