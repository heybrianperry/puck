# Adopt Map-Based Caching for Component Measurement and Reference Storage: Map Entries Cleaned

These rules are ALWAYS ACTIVE for all TypeScript and React files in the codebase that implement component measurement caching, DOM reference storage, URL query parameter caching, and virtualizer handle registration.

### Rules

- **R-MAP-001** MUST: Map entries MUST be cleaned up using Map.delete() when components unmount or are removed from the virtualized list.

### Verify

```bash
# Detect Map.set/get/delete operations
grep -r 'Map\.set\|Map\.get\|Map\.delete' --include='*.tsx' --include='*.ts' packages/core/components/

# Detect URLSearchParams usage
grep -r 'URLSearchParams.*\.get' --include='*.tsx' apps/demo/

# Detect useRef Map instantiation patterns
grep -r 'useRef.*new Map' --include='*.tsx' --include='*.ts' packages/
```

**Accept when:**
- All component measurement storage uses Map.get/set/delete operations with component identifiers as keys
- Every Map.set() call has a corresponding Map.delete() in a useEffect cleanup function
- URL query parameter access uses URLSearchParams.get() for configuration caching
- Map instances are stored in useRef to prevent re-creation on every render
- Component-scoped identifiers are used as Map keys to ensure isolation

<enforcement>
Clause Code MUST NOT skip or defer verification. All Map operations detected in the codebase MUST include corresponding cleanup in useEffect return functions. CI build MUST fail if Map.set operations are detected without cleanup functions. Code review MUST block merge if Map usage does not follow ref storage pattern.
</enforcement>