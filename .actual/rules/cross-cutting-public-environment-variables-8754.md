# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Public Environment Variables

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible code that references environment variables.

### Rules

- **R-NEXTPUB-001** SHOULD: Public environment variables SHOULD be validated for presence before use, with appropriate fallback behavior.
- **R-NEXTPUB-002** MUST: All client-side environment variable references MUST use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-003** MUST: Sensitive credential patterns (PASSWORD, SECRET, KEY, TOKEN) MUST NOT appear with NEXT_PUBLIC_ prefix in environment files.
- **R-NEXTPUB-004** MUST: Build process MUST complete successfully with all required public variables defined.

### Verify

```bash
# Check for client-side process.env usage without NEXT_PUBLIC_ prefix
grep -r 'process\.env\.' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'node_modules' || echo 'All client-side env vars properly prefixed'

# Scan for suspicious public secrets
grep -r 'NEXT_PUBLIC_.*\(PASSWORD\|SECRET\|KEY\|TOKEN\)' . --include='.env*' || echo 'No suspicious public secrets found'

# Verify build completes without environment variable warnings
npm run build 2>&1 | grep -i 'environment variable' || echo 'Build completed without env warnings'
```

**Accept when:**
- All client-side environment variable references use NEXT_PUBLIC_ prefix
- No sensitive credential patterns (PASSWORD, SECRET, private API_KEY) appear with NEXT_PUBLIC_ prefix in environment files
- Build process completes successfully with all required public variables defined
- TypeScript type definitions exist for process.env.NEXT_PUBLIC_* variables
- .env.example file documents all NEXT_PUBLIC_ variables with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side environment variable usage in Next.js applications.
</enforcement>