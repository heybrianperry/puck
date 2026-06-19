# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Public Environment Variables

These rules are ALWAYS ACTIVE for all Next.js client-side components and browser-accessible code requiring runtime configuration in this project.

### Rules

- **R-PUBENV-001** MUST: Public environment variables MUST be accessed via `process.env.NEXT_PUBLIC_*` syntax in client-side code.
- **R-PUBENV-002** MUST: Sensitive credentials (API keys, secrets, passwords, tokens, database connection strings) MUST NOT be prefixed with `NEXT_PUBLIC_`.
- **R-PUBENV-003** MUST: Server-side only environment variables (without `NEXT_PUBLIC_` prefix) MUST NOT be referenced in client-side component files.
- **R-PUBENV-004** SHOULD: Optional public environment variables SHOULD use conditional rendering patterns (e.g., `process.env.NEXT_PUBLIC_VAR && <Component />`) to prevent runtime errors when configuration is missing.
- **R-PUBENV-005** SHOULD: All `NEXT_PUBLIC_` variables SHOULD be documented in `.env.example` and `.env.local.example` files with descriptions and example values.

### Verify

```bash
# Check for proper NEXT_PUBLIC_ usage in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'

# Verify no sensitive patterns in NEXT_PUBLIC_ variable names
grep -r 'NEXT_PUBLIC_.*\(API_KEY\|SECRET\|PASSWORD\|TOKEN\)' apps/docs --include='*.tsx' --include='*.ts' && echo 'FAIL: Sensitive patterns found' || echo 'PASS: No sensitive patterns'

# Count non-NEXT_PUBLIC_ environment variable references in client code (should be minimal)
grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l

# Verify build completes successfully with public env vars
npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'Build validation complete'
```

**Accept when:**
- All client-side environment variable references use `NEXT_PUBLIC_` prefix
- No sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in `NEXT_PUBLIC_` variable names
- Server-side only environment variables are not referenced in client-side component files
- Build process completes successfully with all required `NEXT_PUBLIC_` variables defined
- Optional public variables use conditional rendering to handle missing configuration gracefully

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side code in Next.js applications. Violations MUST be caught during code review and CI pipeline checks before merge.
</enforcement>