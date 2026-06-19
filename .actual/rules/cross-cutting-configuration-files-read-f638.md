# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Configuration Files Read

These rules are ALWAYS ACTIVE for configuration files in documentation applications using Next.js with Nextra framework and Vercel deployment integration.

### Rules

- **R-CONFIG-001** MUST: Configuration files MUST read environment variables from process.env for deployment-specific settings.

### Verify

```bash
# Verify environment variable usage in next.config.mjs
grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs

# Verify canary detection variable
grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs

# Verify Nextra dependency
grep -r '"nextra"' apps/docs/package.json
```

**Accept when:**
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- next.config.mjs file exists and contains configuration logic that reads from process.env
- package.json declares nextra as a dependency in the docs application

<enforcement>
Claude Code MUST NOT skip or defer verification. Build-time validation in next.config.mjs MUST fail if required variables are undefined. CI pipeline MUST fail if verification commands do not find required patterns.
</enforcement>