# Use process.env for Runtime Configuration in Client Components: Runtime Configuration Values

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that require runtime configuration values varying between deployment environments.

### Rules

- **R-RUNTIME-CONFIG-001** MUST: Runtime configuration values that vary between environments MUST be sourced from process.env with the NEXT_PUBLIC_ prefix for client-side access.

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
- No process.env access without NEXT_PUBLIC_ prefix is present in client components

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All three verify commands must execute successfully before accepting changes to client-side configuration patterns.
</enforcement>