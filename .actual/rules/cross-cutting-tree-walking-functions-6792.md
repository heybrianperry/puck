# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Tree Walking Functions

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, state reducer interceptors that wrap core reduction logic, tree-walking operations over ComponentData and RootData structures, and middleware functions that accept user-defined callbacks for data transformation.

### Rules

- **R-TREE-001** MUST: Tree-walking functions MUST provide context information (parentId, propName) to callbacks to enable location-aware transformations.
- **R-TREE-002** MUST: All public API functions that traverse hierarchical data MUST accept callback parameters with context information.
- **R-TREE-003** SHOULD: Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-TREE-004** SHOULD: For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-TREE-005** SHOULD: Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-TREE-006** SHOULD: Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-TREE-007** MAY: Performance-critical paths may bypass callback overhead with explicit documentation of the tradeoff.
- **R-TREE-008** MAY: Internal utility functions not exposed as public API are exempt from callback-based middleware requirements.

### Verify

```bash
# Verify tree-walking functions with generic callback signatures
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
- Callbacks receive correct context information (parentId, propName) and return values are handled properly
- New tree-walking or state management functions follow the callback-based middleware pattern

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review must verify that new tree-walking or state management functions follow the callback-based middleware pattern. Documentation updates are required when new middleware patterns are introduced that deviate from established conventions.
</enforcement>