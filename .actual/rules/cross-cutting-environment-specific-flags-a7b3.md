# Use Environment Variables for Runtime Configuration in External API Clients: Environment Specific Flags

These rules are ALWAYS ACTIVE for React components making external API calls in Next.js applications that require runtime configuration for base URLs and deployment environment flags.

### Rules

- **R-ENV-001** SHOULD: Environment-specific flags (IS_CANARY, IS_LATEST) SHOULD be exposed via NEXT_PUBLIC_ prefixed variables when they affect client-side behavior.
- **R-ENV-002** MUST: All external API fetch calls MUST use environment variables for base URL construction rather than hardcoded URLs.
- **R-ENV-003** MUST: All NEXT_PUBLIC_ environment variables MUST be documented in .env.example with descriptions and example values.
- **R-ENV-004** MUST: All external API failures MUST be logged with console.error including error details for observability.
- **R-ENV-005** SHOULD: A centralized configuration module SHOULD be created to read and validate all NEXT_PUBLIC_ variables at application initialization.
- **R-ENV-006** SHOULD: TypeScript types SHOULD be defined for configuration values to provide compile-time checking where possible.

### Verify

```bash
# Check for NEXT_PUBLIC_ environment variable usage in fetch calls
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules

# Verify fetch calls use BASE_URL from environment
grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules

# Confirm console.error logging for API failures
grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'
```

**Accept when:**
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details
- No hardcoded URLs are detected in fetch calls within React components
- Configuration module validates required NEXT_PUBLIC_ variables at initialization

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>