# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Environment Variable Access

These rules are ALWAYS ACTIVE for Next.js configuration files and runtime code that requires environment-based configuration in the documentation application.

### Rules

- **R-ENV-001** SHOULD: Environment variable access SHOULD occur in next.config.mjs or equivalent configuration entry points.
- **R-ENV-002** MUST: Client-side environment variables MUST use the NEXT_PUBLIC_ prefix to enable access in browser JavaScript.
- **R-ENV-003** SHOULD: Vercel platform variables (VERCEL_GIT_COMMIT_REF) SHOULD be accessed via process.env in configuration files.
- **R-ENV-004** MUST: Hardcoded configuration values MUST NOT be committed to source code; use process.env for environment-specific values.
- **R-ENV-005** SHOULD: All required environment variables SHOULD be documented in README.md with descriptions and example values.

### Verify

```bash
# Check for process.env references in next.config.mjs
grep -r 'process\.env\.' apps/docs/next.config.mjs

# Check for NEXT_PUBLIC_ prefix usage
grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs

# Check for Vercel platform variable access
grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/
```

**Accept when:**
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env
- No hardcoded environment-specific configuration values exist in source code
- Required environment variables are documented in project README

<enforcement>
Clause Code MUST NOT skip or defer verification of environment variable access patterns. All process.env usage must be reviewed for proper scoping (server vs. client) and all client-side variables must use the NEXT_PUBLIC_ prefix.
</enforcement>