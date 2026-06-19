# Validate JSON.parse Input in React Hooks for localStorage Operations: Json Parse Operations

These rules are ALWAYS ACTIVE for all React hooks and utility functions that parse JSON from localStorage, particularly in packages/core/lib/use-sidebar-resize.ts, apps/demo/lib/use-demo-data.ts, and any similar patterns across the codebase.

### Rules

- **R-STORAGE-001** MUST: All JSON.parse operations on localStorage data MUST be wrapped in try-catch blocks to handle malformed input.
- **R-STORAGE-002** MUST: Create and use a shared utility function parseLocalStorageJSON(key, fallback) that encapsulates try-catch and logging logic for all localStorage parsing operations.
- **R-STORAGE-003** MUST: All localStorage parsing errors MUST be logged with sufficient context including the storage key, operation type, and error details for debugging.
- **R-STORAGE-004** SHOULD: Document expected localStorage schema for each hook in JSDoc comments to aid future validation implementation.
- **R-STORAGE-005** SHOULD: Implement comprehensive unit tests for localStorage parsing utilities covering malformed input scenarios (null, undefined, wrong type, corrupted JSON).

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
- All JSON.parse operations on localStorage are wrapped in try-catch or use validated parsing utility
- A shared parseLocalStorageJSON utility function exists and is used consistently across all hooks
- Unit tests exist verifying hooks handle malformed localStorage data (null, undefined, corrupted JSON) without throwing exceptions
- Error logging captures localStorage parse failures with sufficient context for debugging
- grep verification shows zero unvalidated JSON.parse operations on localStorage data

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on localStorage data must be validated before merge. Code review MUST check for compliance with R-STORAGE-001 through R-STORAGE-005. CI pipeline MUST fail if unvalidated JSON.parse operations on localStorage are detected.
</enforcement>