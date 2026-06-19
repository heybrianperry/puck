# Use Environment Variables for Runtime Configuration in External API Clients: Call Failures Logged

These rules are ALWAYS ACTIVE for React components and Next.js client-side code that makes external HTTP requests via fetch, requiring runtime configuration for API endpoints and deployment environment flags.

### Rules

- **R-ENV-001** MUST: API call failures MUST be logged using console.error with descriptive error messages including the error details.
- **R-ENV-002** MUST: All external API fetch calls MUST use environment variables with the NEXT_PUBLIC_ prefix for base URL construction.
- **R-ENV-003** MUST: All required NEXT_PUBLIC_ environment variables MUST be documented in .env.example with descriptions and example values.
- **R-ENV-004** SHOULD: Create a centralized configuration module that reads and validates all NEXT_PUBLIC_ variables at application initialization.
- **R-ENV-005** SHOULD: Use TypeScript to define types for configuration values and provide compile-time checking where possible.
- **R-ENV-006** MAY: Consider creating a custom hook (e.g., useApiClient) that encapsulates environment variable access and fetch logic with consistent error handling.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in fetch calls
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Check for fetch calls using BASE_URL
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Check for console.error logging of API failures
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details
- No hardcoded URLs are detected in fetch calls

<enforcement>
Claude Code MUST NOT skip or defer verification. All external API clients MUST follow the environment variable pattern with NEXT_PUBLIC_ prefix and MUST log failures via console.error.
</enforcement>