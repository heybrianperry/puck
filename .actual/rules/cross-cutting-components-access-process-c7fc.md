# Use Environment Variables for Runtime Configuration in External API Clients: Components Access Process

These rules are ALWAYS ACTIVE for React components in Next.js applications that make external HTTP requests via fetch and require runtime configuration for API endpoints.

### Rules

- **R-ENV-001** MAY: Components MAY access `process.env` directly for `NEXT_PUBLIC_` prefixed variables as they are replaced at build time and available in the browser runtime.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in fetch calls
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Verify fetch calls use BASE_URL from environment variables
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Confirm error logging for external API failures
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All `NEXT_PUBLIC_` variables are documented in `.env.example` with descriptions and example values
- All external API failures are logged with `console.error` including error details
- No hardcoded URLs are present in fetch calls within components
- Configuration module validates required `NEXT_PUBLIC_` variables at application initialization

<enforcement>
Claude Code MUST NOT skip or defer verification. All external API clients in React components MUST use `NEXT_PUBLIC_` prefixed environment variables for runtime configuration. Code review and CI pipeline checks MUST enforce this pattern before merge.
</enforcement>