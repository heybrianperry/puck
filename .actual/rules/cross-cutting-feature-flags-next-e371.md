# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Feature Flags Next

These rules are ALWAYS ACTIVE for Next.js configuration files, runtime configuration access via process.env, and feature flags controlling application behavior in the documentation application.

### Rules

- **R-ENV-001** SHOULD: Feature flags (NEXT_PUBLIC_IS_CANARY) SHOULD be exposed to client-side code when runtime behavior depends on them.
- **R-ENV-002** MUST: Access environment variables in next.config.mjs using process.env.VARIABLE_NAME syntax.
- **R-ENV-003** MUST: Prefix client-side variables with NEXT_PUBLIC_ to enable access in browser JavaScript.
- **R-ENV-004** SHOULD: Document all required environment variables in README.md with descriptions and example values.
- **R-ENV-005** SHOULD: Implement a configuration validation module that runs at application startup to fail fast on missing required variables.
- **R-ENV-006** SHOULD: Use TypeScript type definitions to provide autocomplete and type safety for environment variable access.

### Verify

```bash
# Check for process.env references in next.config.mjs
grep -r 'process\.env\.' apps/docs/next.config.mjs

# Check for NEXT_PUBLIC_ prefixed variables
grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs

# Check for Vercel platform variable usage
grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/
```

**Accept when:**
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env
- All required environment variables are documented in README.md
- Configuration validation is implemented at application startup

<enforcement>
Clause Code MUST NOT skip or defer verification. Pull requests with hardcoded configuration values are rejected during code review. CI pipeline warnings are issued for environment variables without NEXT_PUBLIC_ prefix used in client code. Post-deployment monitoring alerts for missing environment variable errors.
</enforcement>