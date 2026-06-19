# Use process.env for Runtime Configuration in Client Components: Components Access Multiple

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that require runtime access to deployment-specific configuration values.

### Rules

- **R-CONFIG-001** MAY: Components MAY access multiple related environment variables (BASE_URL, IS_CANARY, IS_LATEST) to support feature flags and environment-specific behavior.
- **R-CONFIG-002** MUST: All client-side environment variable access MUST use the NEXT_PUBLIC_ prefix pattern to ensure variables are properly exposed to the browser bundle.
- **R-CONFIG-003** SHOULD: Configuration values SHOULD be extracted into named constants rather than accessing process.env inline throughout component logic.
- **R-CONFIG-004** MUST: Error logging for configuration-dependent operations MUST use console.error with descriptive messages.
- **R-CONFIG-005** SHOULD: Configuration access SHOULD be centralized in a dedicated configuration module (e.g., @/core/config) that exports typed constants and validates required variables at module initialization.

### Verify

```bash
# Verify NEXT_PUBLIC_ prefix usage in client components
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | grep -v "node_modules"

# Verify console.error usage for error logging
grep -r "console\.error" apps/docs/components/ | grep -v "node_modules"

# Check for centralized config module
test -f apps/docs/core/config.ts || echo 'Warning: No centralized config module found'
```

**Accept when:**
- All client-side environment variable access uses the NEXT_PUBLIC_ prefix pattern
- Error logging for configuration-dependent operations uses console.error with descriptive messages
- Configuration values are extracted into named constants rather than accessing process.env inline throughout component logic
- A centralized configuration module exists that validates and exports typed configuration constants
- No process.env access without NEXT_PUBLIC_ prefix is detected in client components

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side components accessing environment variables. Violations must be addressed through code review feedback and refactoring to use the NEXT_PUBLIC_ pattern.
</enforcement>