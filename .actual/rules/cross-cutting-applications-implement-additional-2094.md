# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Applications Implement Additional

These rules are ALWAYS ACTIVE for all React components and TypeScript files in the demo application that access browser localStorage APIs (localStorage, sessionStorage) for configuration or state persistence.

### Rules

- **R-STORAGE-001** MUST: Wrap all `localStorage.getItem()` calls with `JSON.parse()` and include a nullish coalescing operator (`??`) with a safe fallback value to prevent null/undefined errors.
- **R-STORAGE-002** MUST: Encapsulate `JSON.parse()` calls in try-catch blocks to gracefully handle malformed JSON and prevent runtime exceptions from corrupting the component tree.
- **R-STORAGE-003** SHOULD: Create and reuse a utility function `safeParseLocalStorage(key, fallback)` that encapsulates the try-catch and nullish coalescing pattern for consistency across components.
- **R-STORAGE-004** SHOULD: Define TypeScript interfaces for all localStorage data structures and validate parsed objects against these types using type guards to prevent downstream type errors.
- **R-STORAGE-005** MAY: Implement additional sanitization layers for localStorage data that will be rendered in the DOM or used in dynamic code execution to reduce XSS attack surface.
- **R-STORAGE-006** SHOULD: Add logging or monitoring for localStorage parsing failures to track frequency and patterns of invalid data for debugging and security analysis.

### Verify

```bash
# Verify no unprotected localStorage.getItem calls exist without JSON.parse
grep -r 'localStorage\.getItem' apps/demo --include='*.tsx' --include='*.ts' | grep -v 'JSON\.parse' | grep -v '//' | wc -l | grep -q '^0$'

# Verify all JSON.parse calls on localStorage include nullish coalescing fallback
grep -r 'JSON\.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' | grep -v '??' | wc -l | grep -q '^0$'

# Verify localStorage validation tests exist and pass
npm run test -- --testPathPattern='localStorage.*validation' --passWithNoTests=false
```

**Accept when:**
- All `localStorage.getItem()` calls in the codebase are wrapped with `JSON.parse()` and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage
- Try-catch blocks protect all `JSON.parse()` calls that operate on localStorage data
- TypeScript interfaces are defined for all localStorage data structures

<enforcement>
Claude Code MUST NOT skip or defer verification. All localStorage access patterns MUST comply with R-STORAGE-001 through R-STORAGE-006. ESLint custom rules and code review checklists enforce these requirements in the CI pipeline. Pull requests with localStorage changes require security team review approval. Runtime monitoring alerts on excessive JSON parsing errors from localStorage operations.
</enforcement>