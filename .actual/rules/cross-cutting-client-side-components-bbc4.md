# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Client Side Components

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that require dynamic data retrieval from internal API routes at runtime.

### Rules

- **R-FETCH-001** MUST: Client-side components MUST use the native fetch API to communicate with internal API routes when dynamic data retrieval is required at runtime.
- **R-FETCH-002** MUST: API endpoint URLs MUST be constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration.
- **R-FETCH-003** MUST: Error handling with `console.error` or equivalent logging MUST be present for all fetch operations to capture request failures.
- **R-FETCH-004** SHOULD: Wrap fetch calls in try-catch blocks within useEffect hooks and implement cleanup functions with AbortController to prevent memory leaks.
- **R-FETCH-005** SHOULD: Create a shared constants file for API endpoint paths to avoid string duplication and enable easier refactoring.
- **R-FETCH-006** SHOULD: Document required `NEXT_PUBLIC_*` environment variables in `.env.example` and deployment documentation with clear descriptions of their purpose.
- **R-FETCH-007** SHOULD: Implement a simple loading state and error boundary pattern for components that fetch data to provide consistent user feedback.

### Verify

```bash
# Verify native fetch API usage without external HTTP client libraries
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable configuration for API endpoints
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify error handling on fetch operations
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies (axios, ky, etc.)
- API endpoint URLs are constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration
- Error handling with `console.error` or equivalent logging is present for all fetch operations to capture request failures
- AbortController cleanup is implemented in useEffect hooks to prevent memory leaks on component unmount

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All fetch calls in client-side components must be reviewed against R-FETCH-001 through R-FETCH-007. Pull requests introducing external HTTP client libraries for internal API calls must provide documented justification. Missing error handling or hardcoded API URLs trigger code review feedback requiring remediation before merge.
</enforcement>