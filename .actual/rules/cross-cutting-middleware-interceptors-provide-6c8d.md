# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Middleware Interceptors Provide

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, state reducer interceptors that wrap core reduction logic, tree-walking operations over ComponentData and RootData structures, and middleware functions that accept user-defined callbacks for data transformation.

### Rules

- **R-MIDDLEWARE-001** MAY: Middleware interceptors MAY provide conditional execution logic based on action metadata (recordHistory flag, action type filtering).
- **R-MIDDLEWARE-002** MUST: All public API functions that traverse hierarchical data MUST accept callback parameters with context information (parentId, propName) to enable location-aware transformations.
- **R-MIDDLEWARE-003** MUST: Tree-walking and state management functions MUST use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-MIDDLEWARE-004** MUST: Interceptor functions MUST separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-MIDDLEWARE-005** SHOULD: Documentation SHOULD clearly specify the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-MIDDLEWARE-006** SHOULD: Public API contracts SHOULD maintain type safety across middleware boundaries through generic type parameters (UserConfig, UserGenerics, UserData).
- **R-MIDDLEWARE-007** SHOULD: Consumers SHOULD be provided with TypeScript type guards or utility functions to help narrow generic types within callback implementations.

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
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review MUST verify that new tree-walking or state management functions follow the callback-based middleware pattern. Documentation updates are required when new middleware patterns are introduced that deviate from established conventions.
</enforcement>