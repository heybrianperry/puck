# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Localstorage Getitem Calls

These rules are ALWAYS ACTIVE for all React components and TypeScript/JavaScript files in the demo application that access browser localStorage for configuration or state persistence.

### Rules

- **R-STORAGE-001** MUST: All localStorage.getItem() calls MUST be wrapped with JSON.parse() and provide a fallback value using the nullish coalescing operator (??) to handle null or undefined cases.

### Verify

```bash
# Check for unprotected localStorage.getItem calls without JSON.parse
grep -r 'localStorage\.getItem' apps/demo --include='*.tsx' --include='*.ts' | grep -v 'JSON\.parse' | grep -v '//' | wc -l | grep -q '^0$'

# Check for JSON.parse with localStorage but missing nullish coalescing fallback
grep -r 'JSON\.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' | grep -v '??' | wc -l | grep -q '^0$'

# Run localStorage validation tests
npm run test -- --testPathPattern='localStorage.*validation' --passWithNoTests=false
```

**Accept when:**
- All localStorage.getItem() calls in the codebase are wrapped with JSON.parse() and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage
- ESLint verification passes with no violations for unprotected localStorage access patterns

<enforcement>
Clause MUST NOT skip or defer verification. All localStorage access patterns must be validated before code review approval. Violations block CI pipeline execution.
</enforcement>