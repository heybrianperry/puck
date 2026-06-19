# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Client Side Components

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible code that accesses environment variables.

### Rules

- **R-NEXTPUB-001** MUST: Client-side components accessing environment variables MUST use `process.env.NEXT_PUBLIC_*` syntax exclusively.
- **R-NEXTPUB-002** MUST: Server-side API routes, middleware, and server-only code MUST NOT expose non-prefixed environment variables to client-side code.
- **R-NEXTPUB-003** MUST: No sensitive credential patterns (PASSWORD, SECRET, private API_KEY, TOKEN, DATABASE_URL) MUST appear with the NEXT_PUBLIC_ prefix in environment files.
- **R-NEXTPUB-004** SHOULD: Document all NEXT_PUBLIC_ variables in `.env.example` with descriptions and example values.
- **R-NEXTPUB-005** SHOULD: Add TypeScript declarations for `process.env.NEXT_PUBLIC_*` variables to enable autocomplete and type checking.

### Verify

```bash
# Check for client-side process.env usage without NEXT_PUBLIC_ prefix
grep -r 'process\.env\.' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'node_modules' || echo 'No non-prefixed env vars found in client code'

# Scan for suspicious secret patterns with NEXT_PUBLIC_ prefix
grep -r 'NEXT_PUBLIC_.*\(PASSWORD\|SECRET\|KEY\|TOKEN\)' . --include='.env*' || echo 'No suspicious public secrets found'

# Verify build completes without environment variable warnings
npm run build 2>&1 | grep -i 'environment variable' || echo 'Build completed without env warnings'
```

**Accept when:**
- All client-side environment variable references use the NEXT_PUBLIC_ prefix
- No sensitive credential patterns (PASSWORD, SECRET, private API_KEY, TOKEN) appear with NEXT_PUBLIC_ prefix in environment files
- Build process completes successfully with all required public variables defined
- TypeScript type definitions exist for expected NEXT_PUBLIC_* variables
- `.env.example` documents all public environment variables with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Pre-commit hooks and CI pipeline checks MUST validate NEXT_PUBLIC_ prefix usage. Build-time validation MUST fail if client-side code references non-NEXT_PUBLIC_ environment variables or if required public variables are missing. Pull requests MUST be blocked until environment variable usage follows this prefix convention.
</enforcement>