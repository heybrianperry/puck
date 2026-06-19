# Standardize Map-Based Service Definition Lifecycle Management: Service Definitions Stored

These rules are ALWAYS ACTIVE for React-based UI components that use Map data structures to manage service definitions, virtualizers, measurement references, and component-specific lifecycle-managed resources.

### Rules

- **R-SVCDEF-001** MUST: Service definitions MUST be stored in Map data structures using component identifiers or compound keys as lookup keys.
- **R-SVCDEF-002** MUST: Map registries MUST be wrapped in useRef to prevent recreation on every render.
- **R-SVCDEF-003** MUST: All Map.set() operations MUST be paired with corresponding Map.delete() operations in useEffect cleanup functions.
- **R-SVCDEF-004** SHOULD: Use compound keys (e.g., `${zoneId}:${componentId}`) for nested component hierarchies to prevent ID collisions.
- **R-SVCDEF-005** SHOULD: Extract URLSearchParams once at component mount and store in state if dynamic updates are needed.

### Verify

```bash
# Count Map.set() operations across service definition registries
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
- All Map registries include explicit lifecycle management in useEffect cleanup functions.
- Compound keys are used consistently for nested component hierarchies to prevent collisions.

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint errors block CI pipeline for Map set operations without paired delete in cleanup functions. Code review rejection applies to components with Map-based registries lacking explicit lifecycle management. Development console warnings trigger when Map registries exceed size thresholds indicating potential leaks.
</enforcement>