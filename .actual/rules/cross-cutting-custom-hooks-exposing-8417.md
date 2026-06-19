# Validate JSON.parse Input in React Hooks for localStorage Operations: Custom Hooks Exposing

These rules are ALWAYS ACTIVE for React custom hooks that parse JSON from localStorage, particularly those exposing public contracts like useSidebarResize and useDemoData.

### Rules

- **R-HOOKS-001** SHOULD: Custom hooks exposing public contracts (useSidebarResize, useDemoData) SHOULD validate parsed data structure matches expected schema before using the parsed value.
- **R-HOOKS-002** MUST: All JSON.parse operations on localStorage data MUST be wrapped in try-catch blocks with appropriate error handling and logging.
- **R-HOOKS-003** SHOULD: Create and use a shared utility function parseLocalStorageJSON(key, fallback) that encapsulates try-catch and logging logic for all localStorage parsing operations.
- **R-HOOKS-004** SHOULD: Document expected localStorage schema for each hook in JSDoc comments to aid validation implementation and future maintenance.
- **R-HOOKS-005** MUST: Unit tests MUST verify hooks handle malformed JSON, null values, and missing localStorage keys gracefully without throwing exceptions.

### Verify

```bash
# Count unvalidated JSON.parse operations on localStorage
grep -r 'JSON\.parse.*localStorage' --include='*.ts' --include='*.tsx' | grep -v 'try' | wc -l

# Count validated parsing utility usage
grep -r 'parseLocalStorageJSON\|safeParseJSON' --include='*.ts' --include='*.tsx' | wc -l

# Run localStorage error handling tests
npm test -- --testPathPattern='use-.*\.test' --testNamePattern='localStorage.*invalid'
```

**Accept when:**
- All JSON.parse operations on localStorage are wrapped in try-catch or use validated parsing utility
- Unit tests exist verifying hooks handle malformed localStorage data without throwing exceptions
- Error logging captures localStorage parse failures with sufficient context for debugging
- Expected localStorage schema is documented in JSDoc comments for each hook
- A shared parseLocalStorageJSON utility function is implemented and used consistently across all custom hooks

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on localStorage data must be validated before use. Code review MUST block merges lacking error handling for localStorage operations. ESLint custom rules MUST flag unwrapped JSON.parse on localStorage data.
</enforcement>