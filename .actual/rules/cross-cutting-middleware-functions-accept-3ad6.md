# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Middleware Functions Accept

These rules are ALWAYS ACTIVE for all tree-walking middleware implementations, state reducer interceptors, and public API contracts exported from packages/core that handle hierarchical component data structures.

### Rules

- **R-MW-001** MUST: Middleware functions MUST accept callback functions that receive data items and contextual options (parentId, propName) and return transformed data or null.
- **R-MW-002** MUST: Tree-walking functions MUST use generic type parameters constrained to ComponentData, RootData, or UserData.
- **R-MW-003** MUST: State reducer middleware MUST wrap base reducers and provide onAction hooks for interception.
- **R-MW-004** MUST: Public API contracts (walkTree, StateReducer, createReducer) MUST be exported from packages/core modules.
- **R-MW-005** MUST: Callback functions MUST handle null returns gracefully to support filtering operations.
- **R-MW-006** MUST: State reducer interceptors MUST check action.recordHistory flag before invoking record callbacks to avoid unnecessary history entries.
- **R-MW-007** SHOULD: Use mapFields or similar recursive utilities to process nested slot-based content within walkTree implementations.
- **R-MW-008** SHOULD: Provide comprehensive TypeScript documentation with examples, type aliases for common patterns, and inline comments explaining constraint rationale.
- **R-MW-009** SHOULD: Implement performance benchmarks for walkTree and storeInterceptor, establish performance budgets, and consider memoization for expensive transformations.
- **R-MW-010** SHOULD: Add logging/tracing capabilities to middleware functions and provide development-mode warnings for common misuse patterns.

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
- Tree-walking functions use generic type parameters constrained to ComponentData, RootData, or UserData and accept callback functions with contextual options (parentId, propName)
- State reducer middleware wraps base reducers and provides onAction hooks for interception
- Public API contracts (walkTree, StateReducer, createReducer) are exported from packages/core modules
- Callback functions handle null returns to support filtering operations
- State reducer interceptors check action.recordHistory flag before invoking record callbacks
- TypeScript compiler confirms all generic constraints and function signatures are valid

<enforcement>
Verified by: TypeScript compiler type checking for generic constraints and function signatures.
Verified by: Code review verification that middleware functions follow callback-based pattern.
Verified by: Automated grep-based checks in CI pipeline for public API exports.
Violation handling: TypeScript compilation failures block merge for type constraint violations.
Violation handling: Code review feedback requests refactoring for non-compliant middleware implementations.
Violation handling: CI pipeline failures for missing public API exports require correction before merge.
Exception process: Document architectural rationale for deviation in ADR amendment or new ADR.
Exception process: Obtain approval from core library maintainers for changes to public API contracts.
Exception process: Add inline comments explaining why standard middleware pattern cannot be applied.
Claude Code MUST NOT skip or defer verification.
</enforcement>