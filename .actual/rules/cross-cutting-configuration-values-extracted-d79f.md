# Use process.env for Runtime Configuration in Client Components: Configuration Values Extracted

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that access environment variables or make external API calls dependent on deployment-specific configuration.

### Rules

- **R-CONFIG-001** SHOULD: Configuration values SHOULD be extracted into constants (e.g., BASE_URL) rather than accessing process.env repeatedly throughout the component.
- **R-CONFIG-002** MUST: All client-side environment variable access MUST use the NEXT_PUBLIC_ prefix pattern.
- **R-CONFIG-003** SHOULD: Error logging for configuration-dependent operations SHOULD use console.error with descriptive messages.
- **R-CONFIG-004** MUST: Configuration values MUST be extracted into named constants rather than accessing process.env inline throughout component logic.

### Verify

```bash
# Check for NEXT_PUBLIC_ prefix usage in client components
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | grep -v "node_modules"

# Check for console.error usage in error handling
grep -r "console\.error" apps/docs/components/ | grep -v "node_modules"

# Verify centralized config module exists
test -f apps/docs/core/config.ts || echo 'Warning: No centralized config module found'
```

**Accept when:**
- All client-side environment variable access uses the NEXT_PUBLIC_ prefix pattern
- Error logging for configuration-dependent operations uses console.error with descriptive messages
- Configuration values are extracted into named constants rather than accessing process.env inline throughout component logic
- A centralized configuration module (e.g., @/core/config) exports typed constants and validates required variables

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All three verify commands MUST pass before accepting changes to client-side configuration patterns.
</enforcement>