# Use process.env for Runtime Configuration in Client Components: Error Conditions Related

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that make external API calls dependent on environment configuration, access deployment-specific base URLs or feature flags, or interact with release APIs.

### Rules

- **R-CONFIG-001** MUST: Error conditions related to configuration-dependent operations MUST be logged using console.error with descriptive error messages.

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

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All three verification commands must pass before accepting changes to client components that depend on environment configuration.
</enforcement>