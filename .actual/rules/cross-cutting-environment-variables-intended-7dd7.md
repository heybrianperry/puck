# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Environment Variables Intended

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible configuration in this project.

### Rules

- **R-ENV-001** MUST: All environment variables intended for client-side access MUST be prefixed with `NEXT_PUBLIC_`.
- **R-ENV-002** MUST: Server-side API routes, middleware, and internal configuration MUST NOT use the `NEXT_PUBLIC_` prefix for sensitive credentials (database connection strings, private API keys, authentication secrets, internal service URLs, admin tokens).
- **R-ENV-003** SHOULD: Create and maintain a `.env.example` file documenting all `NEXT_PUBLIC_` variables with descriptions and example values.
- **R-ENV-004** SHOULD: Add TypeScript declarations for `process.env.NEXT_PUBLIC_*` variables to enable autocomplete and type checking.
- **R-ENV-005** SHOULD: Implement conditional rendering patterns to gracefully handle missing optional public variables in client-side code.
- **R-ENV-006** MAY: Development and local testing environments may use non-production test values for `NEXT_PUBLIC_` variables (EXC-001).

### Verify

```bash
# Check for client-side process.env usage without NEXT_PUBLIC_ prefix
grep -r 'process\.env\.' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'node_modules' || echo 'All client-side env vars properly prefixed'

# Scan for suspicious secret patterns with NEXT_PUBLIC_ prefix
grep -r 'NEXT_PUBLIC_.*\(PASSWORD\|SECRET\|KEY\|TOKEN\)' . --include='.env*' || echo 'No suspicious public secrets found'

# Verify build completes without environment variable warnings
npm run build 2>&1 | grep -i 'environment variable' || echo 'Build completed without env warnings'
```

**Accept when:**
- All client-side environment variable references use the `NEXT_PUBLIC_` prefix
- No sensitive credential patterns (PASSWORD, SECRET, private API_KEY, TOKEN) appear with `NEXT_PUBLIC_` prefix in environment files
- Build process completes successfully with all required public variables defined
- TypeScript type definitions exist for expected `NEXT_PUBLIC_` variables
- `.env.example` documents all public environment variables with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side environment variable usage. Violations block CI builds and pull request merges.
</enforcement>