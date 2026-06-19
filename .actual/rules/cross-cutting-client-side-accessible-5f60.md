# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Client Side Accessible

These rules are ALWAYS ACTIVE for Next.js configuration files, runtime configuration accessed via process.env, and environment variables provided by deployment platforms like Vercel.

### Rules

- **R-ENV-001** MUST: Client-side accessible environment variables MUST use the NEXT_PUBLIC_ prefix.

### Verify

```bash
# Check for process.env references in next.config.mjs
grep -r 'process\.env\.' apps/docs/next.config.mjs

# Check for NEXT_PUBLIC_ prefixed variables
grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs

# Check for Vercel platform variables accessed via process.env
grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/
```

**Accept when:**
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env
- No hardcoded configuration values are present in source code (except local development defaults)

<enforcement>
Clause Code MUST NOT skip or defer verification. All environment variable usage patterns must be validated against the NEXT_PUBLIC_ prefix requirement for client-side accessible configuration.
</enforcement>