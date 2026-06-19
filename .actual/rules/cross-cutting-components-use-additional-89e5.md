# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Components Use Additional

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that communicate with internal API routes.

### Rules

- **R-FETCH-001** MUST: Use native fetch API for all internal API communication without external HTTP client library dependencies (axios, ky, etc.).
- **R-FETCH-002** MUST: Construct API endpoint URLs using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration.
- **R-FETCH-003** MUST: Implement error handling with console.error or equivalent logging for all fetch operations to capture request failures.
- **R-FETCH-004** MUST: Implement AbortController pattern in useEffect cleanup functions to cancel in-flight requests and prevent memory leaks on component unmount.
- **R-FETCH-005** SHOULD: Create a shared constants file for API endpoint paths to avoid string duplication and enable easier refactoring.
- **R-FETCH-006** SHOULD: Implement loading state and error boundary patterns for components that fetch data to provide consistent user feedback.
- **R-FETCH-007** MAY: Components MAY use additional environment variables (NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) to conditionally modify API behavior or endpoint selection.

### Verify

```bash
# Find all fetch calls in components
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable usage
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Check error handling patterns
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"

# Detect external HTTP client imports that violate the rule
grep -r "import.*from.*['\"]\(axios\|ky\|node-fetch\)['\"]" apps/docs/components --include="*.tsx" --include="*.ts"

# Verify AbortController usage in fetch cleanup
grep -r "AbortController" apps/docs/components --include="*.tsx" --include="*.ts"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using `process.env.NEXT_PUBLIC_*` environment variables for base URL configuration
- Error handling with console.error or equivalent logging is present for all fetch operations to capture request failures
- AbortController is implemented in useEffect cleanup functions for all fetch calls to prevent memory leaks
- No external HTTP client libraries (axios, ky, node-fetch) are imported in component files
- API endpoint paths are centralized in a shared constants file where feasible

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All fetch-based API calls MUST comply with R-FETCH-001 through R-FETCH-004 before code review approval. Violations require explicit justification and architecture review approval.
</enforcement>