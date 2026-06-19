# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Interceptor Functions Separate

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, state reducer interceptors that wrap core reduction logic, tree-walking operations over ComponentData and RootData structures, and middleware functions that accept user-defined callbacks for data transformation.

### Rules

- **R-MIDDLEWARE-001** SHOULD: Interceptor functions SHOULD separate cross-cutting concerns (history recording, action logging) from core business logic through wrapper patterns.
- **R-MIDDLEWARE-002** MUST: All public API functions that traverse hierarchical data MUST accept callback parameters with context information (parentId, propName) to enable location-aware transformations.
- **R-MIDDLEWARE-003** SHOULD: Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-MIDDLEWARE-004** SHOULD: For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-MIDDLEWARE-005** MUST: Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-MIDDLEWARE-006** SHOULD: Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-MIDDLEWARE-007** MUST: Maintain type safety across middleware boundaries through generic type parameters (UserConfig, UserGenerics, UserData).
- **R-MIDDLEWARE-008** SHOULD: Document that callbacks should be pure functions or clearly document side effect expectations; consider adding runtime warnings for detected mutations.
- **R-MIDDLEWARE-009** SHOULD: Provide concrete type examples in documentation and create type utility helpers that simplify common use cases.
- **R-MIDDLEWARE-010** SHOULD: Profile tree-walking performance with realistic data sizes and document performance characteristics in API documentation.

### Verify

```bash
# Verify callback-based middleware functions exist in core package
grep -r 'function.*<.*>.*callbackFn' packages/core/lib --include='*.ts' | grep -v test

# Verify public API exports are present
grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core --include='*.ts'

# Verify TypeScript compilation succeeds for middleware functions
tsc --noEmit packages/core/lib/data/walk-tree.ts packages/core/reducer/index.ts
```

**Accept when:**
- All public API functions that traverse hierarchical data accept callback parameters with context information
- TypeScript compilation succeeds without type errors for generic middleware functions
- Exported public API contracts (walkTree, StateReducer) are present in packages/core with callback-based signatures
- Callbacks receive correct context information and return values are handled properly per integration tests
- Documentation clearly describes callback execution order and timing relative to core algorithms

<enforcement>
TypeScript compiler checks MUST enforce callback signature contracts at build time. Code review MUST verify that new tree-walking or state management functions follow the callback-based middleware pattern. Integration tests MUST validate callback behavior. TypeScript compilation failures MUST block merge for functions that violate generic type constraints. Violations detected in code review MUST request refactoring of direct data mutations to use callback-based transformation.
</enforcement>