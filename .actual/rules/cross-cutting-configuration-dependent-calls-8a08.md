# Use process.env for Runtime Configuration in Client Components: Configuration Dependent Calls

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that make external API calls dependent on environment configuration, particularly within the apps/docs directory.

### Rules

- **R-CONFIG-001** MUST: Configuration-dependent API calls MUST use environment variables to construct endpoint URLs rather than hardcoded values.
- **R-CONFIG-002** MUST: All client-side environment variable access MUST use the NEXT_PUBLIC_ prefix pattern.
- **R-CONFIG-003** MUST: Error logging for configuration-dependent operations MUST use console.error with descriptive messages.
- **R-CONFIG-004** SHOULD: Configuration values SHOULD be extracted into named constants rather than accessing process.env inline throughout component logic.
- **R-CONFIG-005** SHOULD: Extract environment variable access into a dedicated configuration module (e.g., @/core/config) that exports typed constants and validates required variables at module initialization.

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
- TypeScript compilation succeeds with properly typed configuration constants
- No direct process.env access without NEXT_PUBLIC_ prefix is found in client components

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side components making configuration-dependent API calls.
</enforcement>