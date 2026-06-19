# Adopt Map-Based Caching for Component Measurement and Reference Storage: Component Reference Storage

These rules are ALWAYS ACTIVE for all React component files using virtualized lists, drag-and-drop interactions, dynamic layouts, and client-side configuration caching that require efficient storage and retrieval of component measurements, DOM references, and URL query parameters.

### Rules

- **R-MAPREF-001** MUST: Component reference storage MUST use Map.get() and Map.set() operations for retrieval and storage of DOM measurement references.
- **R-MAPREF-002** MUST: Store Map instances in useRef to prevent re-creation on every render: `const mapRef = useRef(new Map())`.
- **R-MAPREF-003** MUST: Always pair Map.set() calls with corresponding Map.delete() calls in useEffect cleanup functions.
- **R-MAPREF-004** MUST: Use componentId or similar unique identifiers as Map keys to prevent collisions.
- **R-MAPREF-005** SHOULD: For URLSearchParams caching, instantiate once and reuse: `const params = new URL(window.location.href).searchParams`.

### Verify

```bash
# Detect Map.set/get/delete operations in component files
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Verify URLSearchParams usage for query parameter caching
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Verify useRef Map instantiation pattern
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

# Verify Map.delete cleanup in useEffect returns
grep -A 5 'Map\.set' --include='*.tsx' --include='*.ts' packages/ | grep -B 5 'useEffect.*cleanup'
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are stored in useRef and not recreated on render cycles
- Component-scoped identifiers prevent collisions between concurrent Map updates

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if Map usage does not follow ref storage pattern. CI build MUST fail if Map operations are detected without cleanup functions. Runtime warnings MUST be emitted in development mode when Map size exceeds thresholds.
</enforcement>