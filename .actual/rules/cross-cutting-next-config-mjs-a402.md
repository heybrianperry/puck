# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Next Config Mjs

These rules are ALWAYS ACTIVE for the `next.config.mjs` file in the documentation application and related configuration files that manage environment-aware settings for Nextra and Vercel integration.

### Rules

- **R-NEXTRA-001** MUST: The next.config.mjs file MUST declare Nextra as a core dependency via package.json reference.
- **R-NEXTRA-002** MUST: The next.config.mjs file MUST include validation logic to check for required environment variables (VERCEL_GIT_COMMIT_REF and NEXT_PUBLIC_IS_CANARY) and provide meaningful error messages when they are missing.
- **R-NEXTRA-003** MUST: All environment variables used in next.config.mjs MUST be documented in README or .env.example file with descriptions of their purpose and expected values.
- **R-NEXTRA-004** SHOULD: Use TypeScript type definitions for process.env to catch missing variables at build time.
- **R-NEXTRA-005** SHOULD: Implement local development setup that simulates Vercel environment variables for consistent testing.

### Verify

```bash
# Verify Nextra dependency is declared
grep -r '"nextra"' apps/docs/package.json

# Verify VERCEL_GIT_COMMIT_REF environment variable usage
grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs

# Verify NEXT_PUBLIC_IS_CANARY environment variable usage
grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs

# Verify next.config.mjs exists and contains configuration logic
test -f apps/docs/next.config.mjs && grep -q 'process\.env' apps/docs/next.config.mjs
```

**Accept when:**
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- next.config.mjs file exists and contains configuration logic that reads from process.env
- package.json declares nextra as a dependency in the docs application
- Validation logic is present in next.config.mjs to handle missing environment variables
- Environment variables are documented in project documentation or .env.example

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for next.config.mjs modifications. CI pipeline MUST fail if verification commands do not find required patterns. Build process MUST terminate with clear error message if environment variables are missing or improperly configured.
</enforcement>