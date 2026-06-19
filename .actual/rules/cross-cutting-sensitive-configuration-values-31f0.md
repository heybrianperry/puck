# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Sensitive Configuration Values

These rules are ALWAYS ACTIVE for Next.js configuration files, runtime configuration accessed via process.env, and environment variables provided by deployment platforms.

### Rules

- **R-NEXTJS-ENV-001** MUST NOT: Sensitive configuration values MUST NOT use the NEXT_PUBLIC_ prefix to prevent client-side exposure.

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
- Sensitive configuration values do not use the NEXT_PUBLIC_ prefix

<enforcement>
Claude Code MUST NOT skip or defer verification of environment variable patterns in Next.js configuration files. All process.env references must be reviewed to ensure sensitive values are not exposed via NEXT_PUBLIC_ prefix.
</enforcement>