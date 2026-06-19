# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Application Code Not

These rules are ALWAYS ACTIVE for all React components and TypeScript/JavaScript files in the demo application that access browser localStorage APIs (localStorage, sessionStorage) for configuration or state persistence.

### Rules

- **R-STORAGE-001** MUST NOT: Application code MUST NOT directly use localStorage values without parsing and validation in security-sensitive contexts.
- **R-STORAGE-002** MUST: All localStorage.getItem() calls MUST be wrapped with JSON.parse() and include nullish coalescing (??) fallback values.
- **R-STORAGE-003** MUST: JSON.parse() calls on localStorage data MUST be protected with try-catch error handling to gracefully handle malformed JSON.
- **R-STORAGE-004** SHOULD: Define TypeScript interfaces for all localStorage data structures and validate parsed objects against these types using type guards.
- **R-STORAGE-005** SHOULD: Create and reuse a utility function safeParseLocalStorage(key, fallback) that encapsulates the try-catch and nullish coalescing pattern.
- **R-STORAGE-006** SHOULD: Add logging or monitoring for localStorage parsing failures to track frequency and patterns of invalid data.
- **R-STORAGE-007** MAY: Use localStorage only for non-critical UI preferences (theme, layout) where parsing failures can safely default to application defaults without security impact (EXC-001).

### Verify

```bash
# Verify no unprotected localStorage.getItem calls exist
grep -r 'localStorage\.getItem' apps/demo --include='*.tsx' --include='*.ts' | grep -v 'JSON\.parse' | grep -v '//' | wc -l | grep -q '^0$'

# Verify all JSON.parse calls on localStorage include nullish coalescing
grep -r 'JSON\.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' | grep -v '??' | wc -l | grep -q '^0$'

# Run localStorage validation tests
npm run test -- --testPathPattern='localStorage.*validation' --passWithNoTests=false
```

**Accept when:**
- All localStorage.getItem() calls in the codebase are wrapped with JSON.parse() and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage
- Try-catch blocks protect all JSON.parse() calls that operate on localStorage data
- TypeScript interfaces are defined for localStorage data structures and exported from a central types file

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to localStorage access patterns. ESLint custom rules MUST detect unprotected localStorage.getItem() and JSON.parse() patterns. Code review MUST include a checklist item for validation of all browser storage access. Pull requests with localStorage changes MUST receive security team review approval.
</enforcement>