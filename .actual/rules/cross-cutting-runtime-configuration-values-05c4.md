# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Runtime Configuration Values

These rules are ALWAYS ACTIVE for Next.js configuration files and runtime configuration access patterns in the documentation application.

### Rules

- **R-RUNTIME-CONFIG-001** MUST: Runtime configuration values MUST be sourced from process.env in Next.js configuration files.

### Verify

```bash
# Check for process.env usage in next.config.mjs
grep -r 'process\.env\.' apps/docs/next.config.mjs

# Check for NEXT_PUBLIC_ prefix usage in next.config.mjs
grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs

# Check for Vercel platform variable access
grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/
```

**Accept when:**
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env
- Environment variables are documented in README.md with descriptions and example values
- Missing environment variables are validated at application startup

<enforcement>
Claude Code MUST NOT skip or defer verification of process.env usage patterns in Next.js configuration files. All runtime configuration must be sourced from process.env, and client-side variables must use the NEXT_PUBLIC_ prefix convention.
</enforcement>