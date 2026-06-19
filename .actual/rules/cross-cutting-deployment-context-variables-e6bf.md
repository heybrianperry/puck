# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Deployment Context Variables

These rules are ALWAYS ACTIVE for Next.js configuration files, runtime configuration accessed via process.env, and environment variables provided by deployment platforms like Vercel.

### Rules

- **R-DEPLOY-001** MUST: Deployment context variables (VERCEL_GIT_COMMIT_REF) MUST be accessed through process.env
- **R-DEPLOY-002** MUST: Client-side environment variables MUST use the NEXT_PUBLIC_ prefix to enable access in browser JavaScript
- **R-DEPLOY-003** SHOULD: Document all required environment variables in README.md with descriptions and example values
- **R-DEPLOY-004** SHOULD: Implement environment variable validation at application startup to fail fast on missing required variables
- **R-DEPLOY-005** MAY: Use TypeScript type definitions to provide autocomplete and type safety for environment variable access

### Verify

```bash
# Check for process.env usage in next.config.mjs
grep -r 'process\.env\.' apps/docs/next.config.mjs

# Check for NEXT_PUBLIC_ prefix usage
grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs

# Check for Vercel platform variables
grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/
```

**Accept when:**
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env
- Environment variables are not hardcoded in source code (except for local development defaults)
- Configuration values that vary across environments are externalized to process.env

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>