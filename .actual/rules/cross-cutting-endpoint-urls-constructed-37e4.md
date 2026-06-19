# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Endpoint Urls Constructed

These rules are ALWAYS ACTIVE for all client-side React components and internal API client code in Next.js applications that fetch data from internal API routes.

### Rules

- **R-FETCH-001** MUST: API endpoint URLs MUST be constructed using environment variables (process.env.NEXT_PUBLIC_*) to support environment-specific configuration

### Verify

```bash
# Verify fetch API usage in components
grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify environment variable usage for endpoint configuration
grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"

# Verify error handling on fetch operations
grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"
```

**Accept when:**
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using process.env.NEXT_PUBLIC_* environment variables for base URL configuration
- Error handling with console.error or equivalent logging is present for all fetch operations to capture request failures

<enforcement>
Clause Code MUST NOT skip or defer verification of R-FETCH-001. All fetch calls must use environment variables for endpoint construction. Pull requests introducing hardcoded URLs or external HTTP client libraries for internal API calls must be rejected until refactored.
</enforcement>