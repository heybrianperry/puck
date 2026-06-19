# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Configuration Expose Next

These rules are ALWAYS ACTIVE for Next.js configuration files in documentation applications using Nextra and deploying to Vercel, specifically `next.config.mjs` and related environment configuration.

### Rules

- **R-CONFIG-001** MUST: Configuration MUST expose NEXT_PUBLIC_IS_CANARY as a public environment variable for client-side canary detection.

### Verify

```bash
# Verify VERCEL_GIT_COMMIT_REF usage in next.config.mjs
grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs

# Verify NEXT_PUBLIC_IS_CANARY usage in next.config.mjs
grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs

# Verify nextra dependency in package.json
grep -r '"nextra"' apps/docs/package.json
```

**Accept when:**
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- next.config.mjs file exists and contains configuration logic that reads from process.env
- package.json declares nextra as a dependency in the docs application
- next.config.mjs includes validation logic to check for required environment variables and provide meaningful error messages
- All environment variables are documented in README or .env.example file with descriptions of their purpose and expected values

<enforcement>
Claude Code MUST NOT skip or defer verification. Build-time validation in next.config.mjs MUST fail if required variables are undefined. CI pipeline MUST fail if verification commands do not find required patterns. Pull requests MUST be blocked until configuration patterns are corrected.
</enforcement>