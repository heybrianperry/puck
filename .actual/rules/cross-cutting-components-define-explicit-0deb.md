# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Components Define Explicit

These rules are ALWAYS ACTIVE for all React components and TypeScript files in the demo application that access browser localStorage for configuration or state persistence.

### Rules

- **R-STORAGE-001** MUST: All `localStorage.getItem()` calls MUST be wrapped with `JSON.parse()` and include a nullish coalescing operator (`??`) with an explicit fallback value that matches the expected data structure.
- **R-STORAGE-002** MUST: All `JSON.parse()` calls on localStorage data MUST be wrapped in try-catch blocks to handle malformed JSON gracefully.
- **R-STORAGE-003** SHOULD: Components SHOULD define explicit fallback values that match the expected data structure rather than using generic empty objects.
- **R-STORAGE-004** SHOULD: Define TypeScript interfaces for all localStorage data structures and validate parsed objects against these types using type guards.
- **R-STORAGE-005** MAY: Create a utility function `safeParseLocalStorage(key, fallback)` that encapsulates the try-catch and nullish coalescing pattern for reuse across components.

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
- All `localStorage.getItem()` calls in the codebase are wrapped with `JSON.parse()` and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage
- All localStorage data structures have corresponding TypeScript interfaces
- Try-catch blocks protect all JSON.parse operations on localStorage data

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint custom rules MUST detect unprotected localStorage.getItem() and JSON.parse() patterns. Code review MUST require validation of all browser storage access. CI pipeline MUST fail on violations and inject malformed localStorage data to verify graceful degradation.
</enforcement>