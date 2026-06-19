# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Public Middleware Functions

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, state reducer interceptors, tree-walking operations over ComponentData and RootData, and middleware functions that accept user-defined callbacks for data transformation.

### Rules

- **R-MIDDLEWARE-001** SHOULD: Public API middleware functions SHOULD use TypeScript generics to maintain type safety while allowing consumers to customize data shapes.
- **R-MIDDLEWARE-002** MUST: When implementing tree-walking middleware, always provide context information (parentId, propName) to callbacks to enable location-aware transformations.
- **R-MIDDLEWARE-003** SHOULD: Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-MIDDLEWARE-004** SHOULD: For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-MIDDLEWARE-005** MUST: Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-MIDDLEWARE-006** SHOULD: Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-MIDDLEWARE-007** SHOULD: Document that callbacks should be pure functions or clearly document side effect expectations; consider adding runtime warnings for detected mutations.
- **R-MIDDLEWARE-008** SHOULD: Provide concrete type examples in documentation; create type utility helpers that simplify common use cases; add integration tests demonstrating typical usage patterns.
- **R-MIDDLEWARE-009** SHOULD: Profile walkTree performance with realistic data sizes; consider adding memoization or short-circuit options; document performance characteristics in API documentation.

### Verify

```bash
# Verify callback-based middleware functions with generic type parameters
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
TypeScript compiler checks enforce callback signature contracts at build time. Code review verifies that new tree-walking or state management functions follow the callback-based middleware pattern. Integration tests validate callback behavior. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review feedback requests refactoring of direct data mutations to use callback-based transformation. Documentation updates are required when new middleware patterns are introduced. Performance-critical paths may bypass callback overhead with explicit documentation. Internal utility functions not exposed as public API are exempt. Exceptions require architectural review and code comment documentation. Claude Code MUST NOT skip or defer verification.
</enforcement>