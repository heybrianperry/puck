# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Middleware Functions Use

These rules are ALWAYS ACTIVE for all middleware functions, tree-walking operations, and state reducer implementations within packages/core that handle ComponentData, RootData, and UserData structures.

### Rules

- **R-MW-001** MAY: Middleware functions MAY use helper utilities like mapFields to recursively process nested data structures with slot-based content.
- **R-MW-002** MUST: Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options.
- **R-MW-003** MUST: State reducer middleware wraps base reducers and provides onAction hooks for interception.
- **R-MW-004** MUST: Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules.
- **R-MW-005** MUST: Callback functions handle null returns gracefully to support filtering operations.
- **R-MW-006** MUST: State reducer interceptors check action.recordHistory flag before invoking record callbacks to avoid unnecessary history entries.
- **R-MW-007** SHOULD: Use mapFields or similar recursive utilities to process nested slot-based content within walkTree implementations.
- **R-MW-008** SHOULD: Export middleware functions as public API contracts from core library index files to establish clear module boundaries.

### Verify

```bash
# Verify tree-walking functions exist with generic type parameters
grep -r 'function walkTree<' packages/core --include='*.ts' | grep -q 'ComponentData\|RootData\|UserData'

# Verify state reducer interceptor pattern
grep -r 'function storeInterceptor' packages/core --include='*.ts' | grep -q 'reducer.*onAction'

# Verify public API exports
grep -r 'export.*walkTree\|StateReducer\|createReducer' packages/core/index.ts packages/core/reducer/index.ts
```

**Accept when:**
- Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options
- State reducer middleware wraps base reducers and provides onAction hooks for interception
- Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules
- Callback functions gracefully handle null returns for filtering operations
- State reducer interceptors check action.recordHistory flag before invoking callbacks
- mapFields or similar recursive utilities are used for nested slot-based content processing

<enforcement>
Clause MUST NOT skip or defer verification. TypeScript compiler type checking for generic constraints and function signatures is mandatory. Code review verification that middleware functions follow callback-based pattern is mandatory. Automated grep-based checks in CI pipeline for public API exports are mandatory. TypeScript compilation failures block merge for type constraint violations. Code review feedback requests refactoring for non-compliant middleware implementations. CI pipeline failures for missing public API exports require correction before merge.
</enforcement>