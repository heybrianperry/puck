# Use Environment Variables for Runtime Configuration in External API Clients: Components Making External

These rules are ALWAYS ACTIVE for React components making external HTTP requests via fetch in Next.js applications, particularly those constructing API endpoints from environment variables with the NEXT_PUBLIC_ prefix.

### Rules

- **R-EX-001** MUST: Components making external fetch calls MUST construct full URLs by combining environment-sourced base URLs with API paths.
- **R-EX-002** MUST: All NEXT_PUBLIC_ environment variables used in external API clients MUST be documented in .env.example with descriptions and example values.
- **R-EX-003** MUST: All external API failures MUST be logged with console.error including error details for observability.
- **R-EX-004** SHOULD: Create a centralized configuration module that reads and validates all NEXT_PUBLIC_ variables at application initialization.
- **R-EX-005** SHOULD: Use TypeScript to define types for configuration values and provide compile-time checking where possible.
- **R-EX-006** MAY: Consider creating a custom hook (e.g., useApiClient) that encapsulates environment variable access and fetch logic with consistent error handling.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in fetch calls
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Verify fetch calls use BASE_URL from environment
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Confirm error logging for API failures
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details
- No hardcoded URLs are detected in fetch calls within components
- Configuration module validates required NEXT_PUBLIC_ variables at initialization

<enforcement>
Claude Code MUST NOT skip or defer verification. All external API clients MUST follow the environment variable pattern. Code review MUST block merges that violate R-EX-001. CI pipeline MUST fail if hardcoded URLs are detected in fetch calls.
</enforcement>