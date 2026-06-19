# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Client Calls Initiated

These rules are ALWAYS ACTIVE for all client-side React components and internal API client code in Next.js applications that fetch data from internal API routes.

### Rules

- **R-FETCH-001** SHOULD: API client calls SHOULD be initiated within React useEffect hooks to ensure proper lifecycle management and avoid server-side execution conflicts.
- **R-FETCH-002** MUST: All fetch calls to internal API routes MUST use the native fetch API without external HTTP client library dependencies (axios, ky, etc.).
- **R-FETCH-003** MUST: API endpoint URLs MUST be constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration.
- **R-FETCH-004** MUST: Error handling with console.error or equivalent logging MUST be present for all fetch operations to capture request failures.
- **R-FETCH-005** SHOULD: Fetch calls SHOULD be wrapped in try-catch blocks within useEffect hooks and implement cleanup functions with AbortController to prevent memory leaks.
- **R-FETCH-006** SHOULD: API endpoint paths SHOULD be defined in shared constants files to avoid string duplication and enable easier refactoring.
- **R-FETCH-007** SHOULD: Components that fetch data SHOULD implement loading state and error boundary patterns to provide consistent user feedback.

### Verify

```bash
# Verify native fetch API usage without external HTTP client libraries
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable configuration for API endpoints
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify error handling on fetch operations
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"

# Verify no axios or other HTTP client imports in components
grep -r "import.*axios\|import.*ky\|from.*axios\|from.*ky" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration
- Error handling with console.error or equivalent logging is present for all fetch operations to capture request failures
- AbortController cleanup is implemented in useEffect hooks to prevent memory leaks on component unmount
- API endpoint paths are centralized in constants files rather than hardcoded in components

<enforcement>
Claude Code MUST NOT skip or defer verification. All fetch-based API client implementations MUST pass the verify commands and accept criteria before approval. Code review checklist MUST verify fetch API usage and environment variable configuration. ESLint rules MUST detect axios or other HTTP client imports in components. Pull requests introducing external HTTP client libraries for internal API calls MUST provide justification. Missing error handling on fetch calls MUST trigger code review feedback. Hardcoded API URLs without environment variable configuration MUST be flagged and refactored.
</enforcement>