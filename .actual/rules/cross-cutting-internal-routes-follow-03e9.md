# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Internal Routes Follow

These rules are ALWAYS ACTIVE for all client-side React components and internal API route implementations in Next.js applications that require dynamic data fetching from internal API endpoints at runtime.

### Rules

- **R-FETCH-001** SHOULD: Internal API routes SHOULD follow the pattern `/api/{resource}` to maintain consistent endpoint structure.
- **R-FETCH-002** MUST: All fetch calls to internal API routes MUST use the native fetch API without external HTTP client library dependencies (axios, ky, etc.).
- **R-FETCH-003** MUST: API endpoint URLs MUST be constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration.
- **R-FETCH-004** MUST: All fetch operations MUST include error handling with `console.error` or equivalent logging to capture request failures.
- **R-FETCH-005** SHOULD: Fetch calls within `useEffect` hooks SHOULD implement cleanup functions with `AbortController` to prevent memory leaks and state updates on unmounted components.
- **R-FETCH-006** SHOULD: API endpoint paths SHOULD be defined in shared constants files to avoid string duplication and enable easier refactoring.
- **R-FETCH-007** MUST: Required `NEXT_PUBLIC_*` environment variables MUST be documented in `.env.example` and deployment documentation with clear descriptions of their purpose.

### Verify

```bash
# Find all fetch calls in components
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable usage
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Check for error handling on fetch operations
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"

# Detect external HTTP client imports that should not be present
grep -r "import.*from.*['\"]axios['\"]\|import.*from.*['\"]ky['\"]" apps/docs/components --include="*.tsx" --include="*.ts"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration
- Error handling with `console.error` or equivalent logging is present for all fetch operations to capture request failures
- Fetch calls in `useEffect` hooks implement `AbortController` cleanup to prevent memory leaks
- API endpoint paths are defined in shared constants rather than hardcoded strings
- Required environment variables are documented in `.env.example` and deployment guides

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All fetch-based API client implementations MUST be reviewed against R-FETCH-001 through R-FETCH-007 before approval. Violations require justification through architecture review or documented exception process.
</enforcement>