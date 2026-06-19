# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Json Parse Operations

These rules are ALWAYS ACTIVE for all React components and TypeScript files in the demo application that access browser localStorage for configuration or state persistence.

### Rules

- **R-STORAGE-001** MUST: JSON.parse() operations on localStorage data MUST be wrapped in try-catch blocks to handle malformed JSON and prevent uncaught exceptions.
- **R-STORAGE-002** MUST: All localStorage.getItem() calls MUST include a nullish coalescing operator (??) with a safe fallback value (typically an empty object "{}" for configuration data).
- **R-STORAGE-003** SHOULD: Create and reuse a utility function safeParseLocalStorage(key, fallback) that encapsulates the try-catch and nullish coalescing pattern for consistency across components.
- **R-STORAGE-004** SHOULD: Define TypeScript interfaces for all localStorage data structures and validate parsed objects against these types using type guards.
- **R-STORAGE-005** SHOULD: Add logging or monitoring for localStorage parsing failures to track frequency and patterns of invalid data.

### Verify

```bash
# Verify no unprotected localStorage.getItem calls exist without JSON.parse
grep -r 'localStorage\.getItem' apps/demo --include='*.tsx' --include='*.ts' | grep -v 'JSON\.parse' | grep -v '//' | wc -l | grep -q '^0$'

# Verify all JSON.parse calls on localStorage include nullish coalescing fallback
grep -r 'JSON\.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' | grep -v '??' | wc -l | grep -q '^0$'

# Run localStorage validation tests
npm run test -- --testPathPattern='localStorage.*validation' --passWithNoTests=false
```

**Accept when:**
- All localStorage.getItem() calls in the codebase are wrapped with JSON.parse() and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage
- ESLint custom rules pass without violations for unprotected localStorage.getItem() and JSON.parse() patterns
- Code review checklist confirms validation of all browser storage access
- Automated testing in CI pipeline successfully injects malformed localStorage data and verifies graceful degradation

<enforcement>
Clause Code MUST NOT skip or defer verification. All localStorage access patterns MUST comply with R-STORAGE-001 and R-STORAGE-002. ESLint violations for unprotected localStorage access MUST cause CI pipeline failure. Pull requests with localStorage changes MUST receive security team review approval.
</enforcement>