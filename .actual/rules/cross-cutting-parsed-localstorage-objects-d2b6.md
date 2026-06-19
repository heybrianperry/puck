# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Parsed Localstorage Objects

These rules are ALWAYS ACTIVE for all React components and TypeScript files in the demo application that access browser localStorage for configuration or state persistence.

### Rules

- **R-STORAGE-001** SHOULD: Parsed localStorage objects SHOULD be validated against expected schema using type guards or validation libraries before use in application logic.

### Verify

```bash
# Verify all localStorage.getItem calls are wrapped with JSON.parse
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
- Components define TypeScript interfaces for all localStorage data structures
- Parsing failures are logged or monitored for tracking invalid data patterns

<enforcement>
Clause Code MUST NOT skip or defer verification of localStorage access patterns. All violations detected by ESLint rules or CI pipeline checks MUST be resolved before merge. Security team review is mandatory for any pull requests modifying browser storage access.
</enforcement>