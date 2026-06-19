# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Configuration Sources Centralized

These rules are ALWAYS ACTIVE for documentation applications using Next.js with Nextra framework deployed to Vercel, specifically governing environment-aware configuration in next.config.mjs and related configuration files.

### Rules

- **R-CONFIG-001** SHOULD: Configuration sources SHOULD be centralized in next.config.mjs to maintain a single source of truth for environment-aware settings.
- **R-CONFIG-002** MUST: next.config.mjs MUST include validation logic to check for required environment variables (VERCEL_GIT_COMMIT_REF, NEXT_PUBLIC_IS_CANARY) and provide meaningful error messages when they are undefined.
- **R-CONFIG-003** SHOULD: Environment variable dependencies SHOULD be documented in README or .env.example file with descriptions of their purpose and expected values.
- **R-CONFIG-004** SHOULD: TypeScript type definitions for process.env SHOULD be used to catch missing variables at build time.
- **R-CONFIG-005** SHOULD: Local development setup SHOULD simulate Vercel environment variables for consistent testing across environments.
- **R-CONFIG-006** MUST: Client-side exposure of deployment metadata via NEXT_PUBLIC_ prefix MUST be intentionally documented to clarify that such variables do not expose sensitive information.

### Verify

```bash
# Verify required environment variables are present in next.config.mjs
grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs
grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs

# Verify Nextra dependency is declared
grep -r '"nextra"' apps/docs/package.json

# Verify next.config.mjs exists and contains configuration logic
test -f apps/docs/next.config.mjs && grep -q 'process\.env' apps/docs/next.config.mjs
```

**Accept when:**
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- next.config.mjs file exists and contains configuration logic that reads from process.env
- package.json declares nextra as a dependency in the docs application
- Validation logic in next.config.mjs fails with clear error messages if required variables are undefined
- Environment variables are documented in README or .env.example

<enforcement>
Claude Code MUST NOT skip or defer verification. Build-time validation in next.config.mjs MUST fail if required variables are undefined. CI pipeline MUST fail if verification commands do not find required patterns. Pull requests MUST be blocked until configuration patterns are corrected.
</enforcement>