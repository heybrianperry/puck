# Adopt Map-Based Caching for Component Measurement and Reference Storage: Component Measurement Data

These rules are ALWAYS ACTIVE for all React component files, virtualization implementations, and client-side configuration handling that require efficient storage and retrieval of dynamically measured UI element dimensions, DOM references, and URL query parameters.

### Rules

- **R-MEAS-001** MUST: Component measurement data MUST be stored using Map.set() with component identifiers as keys and height values as values.
- **R-MEAS-002** MUST: Every Map.set() call MUST have a corresponding Map.delete() in a useEffect cleanup function.
- **R-MEAS-003** MUST: Map instances MUST be stored in useRef to prevent re-creation on every render.
- **R-MEAS-004** MUST: Component-scoped identifiers MUST be used as Map keys to ensure isolation between components and prevent collisions.
- **R-MEAS-005** SHOULD: URL query parameter access SHOULD use URLSearchParams.get() for configuration caching instead of manual parsing.
- **R-MEAS-006** SHOULD: Development-mode logging for Map operations SHOULD be provided to aid debugging when Map contents are not visible to React DevTools.

### Verify

```bash
# Detect Map.set usage with component identifiers as keys
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Verify URLSearchParams caching patterns
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Confirm useRef Map instantiation pattern
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/

# Verify Map.set has corresponding cleanup
grep -B5 -A5 'Map\.set' --include='*.tsx' --include='*.ts' | grep -A5 'useEffect.*cleanup'
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are instantiated within useRef hooks to prevent re-creation across renders
- Component-scoped identifiers are used as Map keys to prevent collisions

<enforcement>
Claude Code MUST NOT skip or defer verification. All Map-based caching implementations MUST pass the verify commands before acceptance. Code review MUST block merge if Map usage does not follow the ref storage pattern with cleanup functions. CI build MUST fail if Map operations are detected without cleanup functions.
</enforcement>