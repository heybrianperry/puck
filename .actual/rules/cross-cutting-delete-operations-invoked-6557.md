# Standardize Map-Based Service Definition Lifecycle Management: Delete Operations Invoked

These rules are ALWAYS ACTIVE for React-based UI components using Map data structures for service definition lifecycle management, particularly those employing useEffect hooks, URLSearchParams configuration, and component-scoped resource registries.

### Rules

- **R-MAP-001** MUST: Delete operations MUST be invoked during component cleanup phases (useEffect cleanup functions or equivalent) to prevent resource leaks.

### Verify

```bash
# Count Map.set() operations across the codebase
grep -r 'measuredItemHeights\.set\|measureRefsRef\.current\.set\|rootVirtualizers\.set' --include='*.tsx' --include='*.ts' | wc -l

# Count corresponding Map.delete() operations
grep -r 'measuredItemHeights\.delete\|measureRefsRef\.current\.delete\|rootVirtualizers\.delete' --include='*.tsx' --include='*.ts' | wc -l

# Verify URLSearchParams usage patterns
grep -r 'params\.get(' --include='*.tsx' --include='*.ts' | grep -c 'URLSearchParams\|searchParams'
```

**Accept when:**
- The number of Map.set() operations matches the number of corresponding Map.delete() operations within component cleanup functions
- All URLSearchParams.get() calls are preceded by URLSearchParams instantiation or searchParams variable declaration
- Map-based registries are stored in useRef hooks rather than component state or module-level variables
- All Map registries paired with set operations include explicit delete calls in useEffect cleanup functions

<enforcement>
Clause R-MAP-001 verification is mandatory. Static analysis via ESLint custom rules MUST detect Map operations without corresponding cleanup. Code review MUST verify lifecycle management patterns. Runtime leak detection in development builds MUST track Map size growth. ESLint errors MUST block CI pipeline for violations. Development console MUST warn when Map registries exceed size thresholds. Exceptions require architecture team approval with inline LEAK-SAFE annotations.
</enforcement>