# Use Environment Variables for Runtime Configuration in External API Clients: External Calls Initiated

These rules are ALWAYS ACTIVE for React components making external HTTP requests via fetch, configuration affecting client-side API endpoint construction, and environment variables with NEXT_PUBLIC_ prefix in Next.js applications.

### Rules

- **R-EX-001** SHOULD: External API calls SHOULD be initiated within useEffect hooks to ensure proper React lifecycle management.
- **R-EX-002** MUST: All external API fetch calls MUST use environment variables with NEXT_PUBLIC_ prefix for base URL construction.
- **R-EX-003** MUST: All required NEXT_PUBLIC_ environment variables MUST be documented in .env.example with descriptions and example values.
- **R-EX-004** MUST: All external API failures MUST be logged with console.error including error details.
- **R-EX-005** MUST: Hardcoded URLs in fetch calls MUST NOT be used; all external API clients MUST use environment variables for configuration.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in external API calls
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Check for fetch calls using BASE_URL configuration
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Check for error logging in API failure scenarios
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details
- No hardcoded URLs are detected in fetch calls
- External API clients are initialized within useEffect hooks

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if hardcoded URLs are detected in fetch calls. Code review MUST block merge if external API clients do not use environment variables. Production monitoring MUST alert on repeated API failures indicating configuration issues.
</enforcement>