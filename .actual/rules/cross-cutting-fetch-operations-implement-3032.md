# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Fetch Operations Implement

These rules are ALWAYS ACTIVE for all client-side React components and internal API client code in Next.js applications that fetch data from internal API routes.

### Rules

- **R-FETCH-001** MUST: Fetch operations MUST implement error handling with observable logging (console.error) to capture and report request failures.
- **R-FETCH-002** MUST: All fetch calls to internal API routes MUST use the native fetch API without external HTTP client library dependencies (axios, ky, etc.).
- **R-FETCH-003** MUST: API endpoint URLs MUST be constructed using process.env.NEXT_PUBLIC_* environment variables for base URL configuration.
- **R-FETCH-004** SHOULD: Implement AbortController pattern in useEffect cleanup functions for all fetch calls to prevent memory leaks and state updates on unmounted components.
- **R-FETCH-005** SHOULD: Wrap fetch calls in try-catch blocks within useEffect hooks and implement proper cleanup functions.
- **R-FETCH-006** SHOULD: Create shared constants files for API endpoint paths to avoid string duplication and enable easier refactoring.
- **R-FETCH-007** SHOULD: Document required NEXT_PUBLIC_* environment variables in .env.example and deployment documentation with clear descriptions of their purpose.

### Verify

```bash
# Find all fetch calls in components
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable usage
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Check for error handling with console.error
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"

# Detect external HTTP client imports that should not be present
grep -r "import.*from.*['\"]axios['\"]\|import.*from.*['\"]ky['\"]\|import.*from.*['\"]node-fetch['\"]" apps/docs/components --include="*.tsx" --include="*.ts"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using process.env.NEXT_PUBLIC_* environment variables for base URL configuration
- Error handling with console.error or equivalent logging is present for all fetch operations to capture request failures
- AbortController or equivalent cleanup patterns are implemented in useEffect hooks to prevent memory leaks
- No hardcoded API URLs are present in component code

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All fetch operations in internal API client code MUST be reviewed against R-FETCH-001 through R-FETCH-007 before code is considered compliant.
</enforcement>