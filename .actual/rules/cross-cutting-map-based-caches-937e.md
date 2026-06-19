# Adopt Map-Based Caching for Component Measurement and Reference Storage: Map Based Caches

These rules are ALWAYS ACTIVE for all React component files using virtualized lists, drag-and-drop interactions, dynamic layouts, or URL query parameter caching that require efficient storage and retrieval of measurements, DOM references, and configuration parameters.

### Rules

- **R-MAP-001** SHOULD: Map-based caches SHOULD be stored in refs (useRef) to prevent re-renders when cache contents change.
- **R-MAP-002** MUST: Always pair Map.set() calls with corresponding Map.delete() calls in useEffect cleanup functions.
- **R-MAP-003** MUST: Use component-scoped identifiers (componentId or similar unique identifiers) as Map keys to prevent collisions.
- **R-MAP-004** SHOULD: For URLSearchParams caching, instantiate once and reuse via useRef: const params = new URL(window.location.href).searchParams.
- **R-MAP-005** MUST: Store Map instances in useRef to prevent re-creation on every render: const mapRef = useRef(new Map()).

### Verify

```bash
# Detect Map usage patterns in component files
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Verify URLSearchParams caching patterns
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Verify useRef Map instantiation pattern
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

# Verify cleanup functions exist for Map operations
grep -B5 -A5 'Map\.set' --include='*.tsx' --include='*.ts' packages/ | grep -c 'useEffect\|cleanup'
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are stored in useRef and not recreated on render cycles
- Component-scoped identifiers are used consistently as Map keys

<enforcement>
Claude Code MUST NOT skip or defer verification. All Map-based caching patterns MUST comply with these rules before code review approval. CI build MUST fail if Map operations are detected without cleanup functions or if Map usage does not follow ref storage pattern.
</enforcement>