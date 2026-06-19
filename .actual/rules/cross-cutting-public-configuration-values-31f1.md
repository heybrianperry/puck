# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Public Configuration Values

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible code that references environment variables.

### Rules

- **R-NEXTPUB-001** MUST: All environment variables exposed to client-side code MUST be prefixed with `NEXT_PUBLIC_`.
- **R-NEXTPUB-002** MUST: Server-side API routes, middleware, and database connection strings MUST NOT use the `NEXT_PUBLIC_` prefix.
- **R-NEXTPUB-003** MUST: Sensitive credentials (passwords, API keys, tokens, database URLs) MUST NEVER be prefixed with `NEXT_PUBLIC_`.
- **R-NEXTPUB-004** SHOULD: Public configuration values SHOULD be documented in environment variable reference documentation (e.g., `.env.example`).
- **R-NEXTPUB-005** SHOULD: TypeScript type definitions for `process.env.NEXT_PUBLIC_*` variables SHOULD be declared to enable autocomplete and type checking.
- **R-NEXTPUB-006** MAY: Conditional rendering patterns MAY be used to gracefully handle missing optional public variables in client-side code.

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
- All client-side environment variable references use `NEXT_PUBLIC_` prefix
- No sensitive credential patterns (PASSWORD, SECRET, KEY, TOKEN) appear with `NEXT_PUBLIC_` prefix in environment files
- Build process completes successfully with all required public variables defined
- TypeScript type definitions exist for expected `NEXT_PUBLIC_*` variables
- `.env.example` documents all public configuration values with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side environment variable usage in Next.js applications.
</enforcement>