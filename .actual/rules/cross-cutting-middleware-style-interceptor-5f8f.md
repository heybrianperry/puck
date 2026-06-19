# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Middleware Style Interceptor

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, including state reducer interceptors and tree-walking operations over ComponentData, RootData, zones, and content arrays.

### Rules

- **R-MIDDLEWARE-001** MUST: Middleware-style interceptor functions MUST preserve the input data type in the return type, ensuring type safety across transformation pipelines.
- **R-MIDDLEWARE-002** MUST: All public API functions that traverse hierarchical data MUST accept callback parameters with context information (parentId, propName) to enable location-aware transformations.
- **R-MIDDLEWARE-003** SHOULD: Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-MIDDLEWARE-004** SHOULD: For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-MIDDLEWARE-005** SHOULD: Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-MIDDLEWARE-006** SHOULD: Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-MIDDLEWARE-007** MAY: Performance-critical paths may bypass callback overhead with explicit documentation of the tradeoff.
- **R-MIDDLEWARE-008** MAY: Internal utility functions not exposed as public API are exempt from callback-based middleware requirements.

### Verify

```bash
# Verify callback-based middleware signatures in core library
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
Clause MUST NOT skip or defer verification. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review verifies that new tree-walking or state management functions follow the callback-based middleware pattern. Documentation updates are required when new middleware patterns are introduced that deviate from established conventions.
</enforcement>