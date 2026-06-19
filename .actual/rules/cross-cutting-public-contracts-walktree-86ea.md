# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Public Contracts Walktree

These rules are ALWAYS ACTIVE for all files in packages/core that implement or export tree-walking middleware, state reducers, and public API contracts for component data transformation.

### Rules

- **R-WALKTREE-001** MUST: Public API contracts (walkTree, StateReducer, createReducer) MUST be exported from core library modules to establish middleware boundaries.
- **R-WALKTREE-002** MUST: Tree-walking functions MUST use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options.
- **R-WALKTREE-003** MUST: State reducer middleware MUST wrap base reducers and provide onAction hooks for interception.
- **R-WALKTREE-004** SHOULD: Callback functions SHOULD handle null returns gracefully to support filtering operations.
- **R-WALKTREE-005** SHOULD: State reducer interceptors SHOULD check action.recordHistory flag before invoking record callbacks to avoid unnecessary history entries.
- **R-WALKTREE-006** SHOULD: Use mapFields or similar recursive utilities to process nested slot-based content within walkTree implementations.

### Verify

```bash
# Verify tree-walking functions use generic type parameters with proper constraints
grep -r 'function walkTree<' packages/core --include='*.ts' | grep -q 'ComponentData\|RootData\|UserData'

# Verify state reducer middleware pattern is implemented
grep -r 'function storeInterceptor' packages/core --include='*.ts' | grep -q 'reducer.*onAction'

# Verify public API contracts are exported from core modules
grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core/index.ts packages/core/reducer/index.ts
```

**Accept when:**
- Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options
- State reducer middleware wraps base reducers and provides onAction hooks for interception
- Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules
- TypeScript compiler confirms all generic constraints and function signatures are valid
- Middleware functions follow callback-based pattern without embedding transformation logic in data structures

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for type constraint violations. Code review must verify middleware functions follow callback-based pattern. CI pipeline must confirm public API exports are present before merge.
</enforcement>