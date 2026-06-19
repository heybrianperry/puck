# Use process.env for Runtime Configuration in Client Components: Environment Variable Access

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that require runtime access to deployment-specific configuration values.

### Rules

- **R-ENV-001** SHOULD: Environment variable access SHOULD occur at component initialization or within useEffect hooks to ensure values are available at runtime.
- **R-ENV-002** MUST: All client-side environment variable access MUST use the NEXT_PUBLIC_ prefix pattern to ensure variables are available in the browser bundle.
- **R-ENV-003** SHOULD: Configuration values SHOULD be extracted into named constants rather than accessing process.env inline throughout component logic.
- **R-ENV-004** SHOULD: Error logging for configuration-dependent operations SHOULD use console.error with descriptive messages.
- **R-ENV-005** SHOULD: Configuration values SHOULD be extracted into a dedicated configuration module (e.g., @/core/config) that exports typed constants and validates required variables at module initialization.

### Verify

```bash
# Check for NEXT_PUBLIC_ prefixed environment variable usage in client components
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | grep -v "node_modules"

# Check for console.error usage in configuration-dependent operations
grep -r "console\.error" apps/docs/components/ | grep -v "node_modules"

# Verify centralized config module exists
test -f apps/docs/core/config.ts || echo 'Warning: No centralized config module found'
```

**Accept when:**
- All client-side environment variable access uses the NEXT_PUBLIC_ prefix pattern
- Error logging for configuration-dependent operations uses console.error with descriptive messages
- Configuration values are extracted into named constants rather than accessing process.env inline throughout component logic
- A centralized configuration module exists that exports typed constants and validates required variables
- TypeScript types are applied to configuration values to ensure type safety

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All client-side environment variable access patterns MUST conform to the NEXT_PUBLIC_ prefix convention and configuration extraction requirements.
</enforcement>