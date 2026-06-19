# Adopt Map-Based Caching for Component Measurement and Reference Storage: Additional Cache Structures

These rules are ALWAYS ACTIVE for all React component files, virtualization implementations, and client-side configuration handling that require efficient storage and retrieval of dynamically measured UI element dimensions, DOM references, and URL query parameters.

### Rules

- **R-CACHE-001** MUST: Store Map instances in useRef to prevent re-creation on every render: `const mapRef = useRef(new Map())`
- **R-CACHE-002** MUST: Always pair Map.set() calls with corresponding Map.delete() calls in useEffect cleanup functions
- **R-CACHE-003** MUST: Use componentId or similar unique identifiers as Map keys to prevent collisions
- **R-CACHE-004** MUST: Call Map.delete() on component unmount to prevent memory leaks
- **R-CACHE-005** SHOULD: Use URLSearchParams.get() for URL query parameter caching instead of manual parsing
- **R-CACHE-006** SHOULD: Instantiate URLSearchParams once and reuse: `const params = new URL(window.location.href).searchParams`
- **R-CACHE-007** MAY: Use additional cache structures such as Set for tracking indexes or identifiers that require uniqueness guarantees

### Verify

```bash
# Verify Map usage patterns in component files
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Verify URLSearchParams caching patterns
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Verify useRef Map instantiation patterns
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

# Verify Map.set has corresponding cleanup
grep -B5 -A5 'Map\.set' --include='*.tsx' --include='*.ts' packages/ | grep -c 'useEffect.*cleanup'
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are stored in useRef and not recreated on render cycles
- Component-scoped identifiers are used as Map keys to ensure isolation

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST check that all Map.set() operations have corresponding cleanup. ESLint custom rules and runtime memory profiling in CI MUST detect Map growth patterns and missing cleanup functions. Build fails if violations are detected.
</enforcement>