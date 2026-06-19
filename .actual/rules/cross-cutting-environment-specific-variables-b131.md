# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Environment Specific Variables

These rules are ALWAYS ACTIVE for all Next.js configuration files, environment variable declarations, and Nextra-integrated documentation applications, particularly `next.config.mjs` and related configuration sources in the `apps/docs/` directory.

### Rules

- **R-ENV-001** SHOULD: Environment-specific variables SHOULD follow the `NEXT_PUBLIC_` prefix convention for client-side accessibility.
- **R-ENV-002** MUST: Environment variable dependencies in `next.config.mjs` MUST include validation logic and provide meaningful error messages when required variables are undefined.
- **R-ENV-003** MUST: All environment variables used in configuration MUST be documented in README or `.env.example` file with descriptions of their purpose and expected values.
- **R-ENV-004** SHOULD: TypeScript type definitions for `process.env` SHOULD be implemented to catch missing variables at build time.
- **R-ENV-005** SHOULD: Local development setup SHOULD simulate Vercel environment variables for consistent testing across environments.

### Verify

```bash
# Verify required environment variable usage in next.config.mjs
grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs
grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs

# Verify Nextra dependency is declared
grep -r '"nextra"' apps/docs/package.json

# Verify next.config.mjs exists and contains environment-aware logic
test -f apps/docs/next.config.mjs && grep -q 'process\.env' apps/docs/next.config.mjs
```

**Accept when:**
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- `next.config.mjs` file exists and contains configuration logic that reads from `process.env`
- `package.json` declares `nextra` as a dependency in the docs application
- Validation logic is present in `next.config.mjs` to handle undefined environment variables
- Environment variables are documented in project configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting configuration changes. Build process MUST terminate with clear error messages if environment variables are missing or improperly configured. Pull requests MUST be blocked until configuration patterns are corrected.
</enforcement>