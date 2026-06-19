# Validate JSON.parse Input in React Hooks for localStorage Operations: React Hooks That

These rules are ALWAYS ACTIVE for React hooks that parse JSON from localStorage, particularly in packages/core/lib/use-sidebar-resize.ts, apps/demo/lib/use-demo-data.ts, and any custom hooks that interact with browser storage APIs.

### Rules

- **R-STORAGE-001** MUST: React hooks that parse localStorage data MUST provide fallback values when parsing fails.
- **R-STORAGE-002** MUST: All JSON.parse operations on localStorage data MUST be wrapped in try-catch blocks or use a validated parsing utility.
- **R-STORAGE-003** MUST: Error logging for localStorage parse failures MUST capture sufficient context (key name, error message, fallback value used) for debugging.
- **R-STORAGE-004** SHOULD: Create and use a shared utility function parseLocalStorageJSON(key, fallback) that encapsulates try-catch and logging logic across all hooks.
- **R-STORAGE-005** SHOULD: Document expected localStorage schema for each hook in JSDoc comments to aid future validation implementation.

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
- Unit tests exist verifying hooks handle malformed localStorage data without throwing exceptions
- Error logging captures localStorage parse failures with sufficient context for debugging
- No unvalidated JSON.parse operations on localStorage data remain in codebase

<enforcement>
Clause Code MUST NOT skip or defer verification. All localStorage parsing operations must be validated before merge. CI pipeline MUST fail if unvalidated JSON.parse on localStorage is detected. Code review MUST block merge if localStorage parsing lacks error handling.
</enforcement>