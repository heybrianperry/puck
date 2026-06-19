# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: State Reducer Interceptors

These rules are ALWAYS ACTIVE for all tree-walking operations on ComponentData, RootData, and UserData structures; state reducer middleware for action interception and history recording; and generic callback-based transformation functions exported from packages/core.

### Rules

- **R-MIDDLEWARE-001** MUST: State reducer interceptors MUST wrap base reducers and provide hooks for recording state changes and executing onAction callbacks.

### Verify

```bash
# Verify tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData
grep -r 'function walkTree<' packages/core --include='*.ts' | grep -q 'ComponentData\|RootData\|UserData'

# Verify state reducer middleware wraps base reducers with onAction hooks
grep -r 'function storeInterceptor' packages/core --include='*.ts' | grep -q 'reducer.*onAction'

# Verify public API contracts are exported from packages/core modules
grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core/index.ts packages/core/reducer/index.ts
```

**Accept when:**
- Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options
- State reducer middleware wraps base reducers and provides onAction hooks for interception
- Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules
- Callback functions handle null returns gracefully to support filtering operations
- State reducer interceptors check action.recordHistory flag before invoking record callbacks
- Nested slot-based content is processed using mapFields or similar recursive utilities within walkTree implementations

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compiler type checking for generic constraints and function signatures is mandatory. Code review verification that middleware functions follow callback-based pattern is required. Automated grep-based checks in CI pipeline for public API exports must pass before merge.
</enforcement>