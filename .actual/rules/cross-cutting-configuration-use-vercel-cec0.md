# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Configuration Use Vercel

These rules are ALWAYS ACTIVE for configuration files in the documentation application, specifically `next.config.mjs` and related environment-aware setup files that integrate with Nextra and Vercel.

### Rules

- **R-CONFIG-001** MUST: Configuration MUST use VERCEL_GIT_COMMIT_REF to determine the active Git branch context.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. Build-time validation in next.config.mjs MUST fail if required variables are undefined. CI pipeline MUST fail if verification commands do not find required patterns.
</enforcement>