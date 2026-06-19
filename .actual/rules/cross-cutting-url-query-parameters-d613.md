# Adopt Map-Based Caching for Component Measurement and Reference Storage: Url Query Parameters

These rules are ALWAYS ACTIVE for all React component files, virtualization implementations, and client-side configuration code that manages component measurements, DOM references, and URL query parameters.

### Rules

- **R-MAP-001** SHOULD: URL query parameters SHOULD be cached using URLSearchParams.get() for runtime configuration decisions.
- **R-MAP-002** MUST: Store Map instances in useRef to prevent re-creation on every render.
- **R-MAP-003** MUST: Always pair Map.set() calls with corresponding Map.delete() calls in useEffect cleanup functions.
- **R-MAP-004** MUST: Use componentId or similar unique identifiers as Map keys to prevent collisions.
- **R-MAP-005** SHOULD: For URLSearchParams caching, instantiate once and reuse: const params = new URL(window.location.href).searchParams.

### Verify

```bash
# Detect Map usage patterns in component files
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Verify URLSearchParams caching usage
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Identify useRef Map instantiation patterns
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

# Verify cleanup functions exist for Map operations
grep -B5 -A5 'Map\.set' --include='*.tsx' --include='*.ts' | grep -c 'useEffect.*cleanup'
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are stored in useRef and not recreated on render cycles
- Component-scoped identifiers prevent key collisions across different Map instances

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST check that Map cleanup is present in useEffect returns. CI build MUST fail if Map operations are detected without cleanup functions. Runtime memory profiling MUST be performed to detect Map growth patterns.
</enforcement>