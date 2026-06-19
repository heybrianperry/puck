# Use Environment Variables for Runtime Configuration in External API Clients: External Clients Source

These rules are ALWAYS ACTIVE for React components making external HTTP requests via fetch, configuration affecting client-side API endpoint construction, and environment variables with NEXT_PUBLIC_ prefix in Next.js applications.

### Rules

- **R-EX-001** MUST: External API clients MUST source base URLs from environment variables prefixed with NEXT_PUBLIC_ to ensure availability in browser runtime.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in external API clients
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Verify fetch calls use BASE_URL from environment variables
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Confirm error logging for external API failures
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details

<enforcement>
Claude Code MUST NOT skip or defer verification. All external API clients must be inspected to confirm NEXT_PUBLIC_ environment variable usage before accepting changes.
</enforcement>