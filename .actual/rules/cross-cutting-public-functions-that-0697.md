# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Public Functions That

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data structures, including tree-walking operations, state reducers, and middleware interceptors.

### Rules

- **R-MIDDLEWARE-001** MUST: Public API functions that traverse hierarchical data structures MUST accept callback functions as parameters to enable consumer-defined transformation logic.
- **R-MIDDLEWARE-002** MUST: Tree-walking middleware MUST provide context information (parentId, propName) to callbacks to enable location-aware transformations.
- **R-MIDDLEWARE-003** MUST: Callback-based middleware MUST use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-MIDDLEWARE-004** SHOULD: Interceptor functions SHOULD separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-MIDDLEWARE-005** SHOULD: Documentation SHOULD clearly specify the execution order and timing of callbacks relative to the core algorithm.
- **R-MIDDLEWARE-006** SHOULD: Public API contracts SHOULD provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-MIDDLEWARE-007** MAY: Performance-critical paths MAY bypass callback overhead with explicit documentation of the tradeoff and architectural review.

### Verify

```bash
# Verify callback-based middleware patterns in public API
grep -r 'function.*<.*>.*callbackFn' packages/core/lib --include='*.ts' | grep -v test

# Verify public API exports
grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core --include='*.ts'

# Verify TypeScript compilation
tsc --noEmit packages/core/lib/data/walk-tree.ts packages/core/reducer/index.ts
```

**Accept when:**
- All public API functions that traverse hierarchical data accept callback parameters with context information
- TypeScript compilation succeeds without type errors for generic middleware functions
- Exported public API contracts (walkTree, StateReducer) are present in packages/core with callback-based signatures
- Callbacks receive correct context information and return values are handled properly per integration tests

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review MUST verify that new tree-walking or state management functions follow the callback-based middleware pattern. Documentation updates are required when new middleware patterns are introduced that deviate from established conventions.
</enforcement>