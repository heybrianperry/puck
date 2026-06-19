# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Server Side Secrets

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible configuration in this project.

### Rules

- **R-NEXTPUB-001** MUST NOT: Server-side secrets (database credentials, private API keys, authentication tokens) MUST NOT use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-002** MUST: All client-side environment variable references in React components and pages MUST use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-003** MUST: Public-facing configuration (analytics domains, public API endpoints, feature flags) MUST be prefixed with NEXT_PUBLIC_ when accessed from client-side code.
- **R-NEXTPUB-004** SHOULD: Document all NEXT_PUBLIC_ variables in .env.example with descriptions and example values.
- **R-NEXTPUB-005** SHOULD: Add TypeScript declarations for process.env.NEXT_PUBLIC_* variables to enable autocomplete and type checking.

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
- All client-side environment variable references use NEXT_PUBLIC_ prefix
- No sensitive credential patterns (PASSWORD, SECRET, private API_KEY) appear with NEXT_PUBLIC_ prefix in environment files
- Build process completes successfully with all required public variables defined
- TypeScript type definitions exist for NEXT_PUBLIC_* variables
- .env.example documents all public environment variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to environment variable usage or client-side configuration.
</enforcement>