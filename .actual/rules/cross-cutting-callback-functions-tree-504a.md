# Adopt Generic Tree-Walking Middleware for Public API Data Transformation: Callback Functions Tree

These rules are ALWAYS ACTIVE for all public API functions exported from packages/core that traverse or transform hierarchical data, state reducer interceptors that wrap core reduction logic, tree-walking operations over ComponentData and RootData structures, and middleware functions that accept user-defined callbacks for data transformation.

### Rules

- **R-CALLBACK-001** MUST: Callback functions in tree-walking APIs MUST support returning null or void to indicate no transformation, with the original value preserved.
- **R-CALLBACK-002** MUST: All public API functions that traverse hierarchical data MUST accept callback parameters with context information (parentId, propName).
- **R-CALLBACK-003** MUST: Use the null-coalescing pattern (callbackFn(...) ?? originalValue) to allow callbacks to opt out of transformation by returning null or void.
- **R-CALLBACK-004** SHOULD: For interceptor functions, separate concerns by accepting multiple optional callbacks (record, onAction) rather than a single monolithic callback.
- **R-CALLBACK-005** SHOULD: Document the execution order and timing of callbacks relative to the core algorithm (e.g., storeInterceptor calls onAction after reducer completes).
- **R-CALLBACK-006** SHOULD: Provide TypeScript type guards or utility functions to help consumers narrow generic types within callback implementations.
- **R-CALLBACK-007** SHOULD: Document that callbacks should be pure functions or clearly document side effect expectations; consider adding runtime warnings for detected mutations.
- **R-CALLBACK-008** MAY: Performance-critical paths may bypass callback overhead with explicit documentation of the tradeoff.
- **R-CALLBACK-009** MAY: Internal utility functions not exposed as public API are exempt from callback-based middleware requirements.

### Verify

```bash
# Verify callback function signatures in tree-walking APIs
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
- Callback functions support returning null or void to indicate no transformation
- The null-coalescing pattern is used to preserve original values when callbacks opt out

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for functions that violate generic type constraints. Code review must verify that new tree-walking or state management functions follow the callback-based middleware pattern. Integration tests must validate that callbacks receive correct context information and return values are handled properly.
</enforcement>