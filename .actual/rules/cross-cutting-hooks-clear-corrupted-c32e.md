# Validate JSON.parse Input in React Hooks for localStorage Operations: Hooks Clear Corrupted

These rules are ALWAYS ACTIVE for React hooks in `packages/core/lib/use-sidebar-resize.ts`, `apps/demo/lib/use-demo-data.ts`, and any other custom hooks that parse JSON from localStorage.

### Rules

- **R-HOOKS-001** MUST: Wrap all `JSON.parse()` operations on localStorage data in try-catch blocks to prevent runtime exceptions from malformed input.
- **R-HOOKS-002** MUST: Log parse failures with sufficient context (key name, error message, operation type) to enable debugging and monitoring.
- **R-HOOKS-003** MAY: Clear corrupted localStorage entries after detecting parse failures to prevent repeated errors on subsequent hook invocations.
- **R-HOOKS-004** SHOULD: Provide and use a shared utility function `parseLocalStorageJSON(key, fallback)` that encapsulates validation, error handling, and logging logic.
- **R-HOOKS-005** MUST: Implement unit tests verifying hooks handle malformed JSON, null values, undefined, and missing localStorage keys without throwing exceptions.
- **R-HOOKS-006** SHOULD: Document expected localStorage schema for each hook in JSDoc comments to aid future validation implementation and maintenance.

### Verify

```bash
# Count unvalidated JSON.parse operations on localStorage
grep -r 'JSON\.parse.*localStorage' --include='*.ts' --include='*.tsx' | grep -v 'try' | wc -l

# Count validated parsing utility usage
grep -r 'parseLocalStorageJSON\|safeParseJSON' --include='*.ts' --include='*.tsx' | wc -l

# Verify localStorage error handling tests exist
npm test -- --testPathPattern='use-.*\.test' --testNamePattern='localStorage.*invalid'
```

**Accept when:**
- All `JSON.parse()` operations on localStorage are wrapped in try-catch blocks or use a validated parsing utility function.
- Unit tests exist verifying hooks handle malformed localStorage data, null values, and missing keys without throwing exceptions.
- Error logging captures localStorage parse failures with key name, error message, and operation context.
- Shared utility function `parseLocalStorageJSON()` or equivalent is implemented and used consistently across all localStorage-parsing hooks.
- JSDoc comments document expected localStorage schema and fallback behavior for each hook.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval. Violations must be caught during CI pipeline execution and code review before merge.
</enforcement>