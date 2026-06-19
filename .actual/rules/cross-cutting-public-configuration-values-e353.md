# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Public Configuration Values

These rules are ALWAYS ACTIVE for all Next.js client-side components, pages, and browser-accessible code that requires runtime configuration values.

### Rules

- **R-PUBCONF-001** SHOULD: Public configuration values SHOULD be documented in environment variable templates or README files.
- **R-PUBCONF-002** MUST: All client-side environment variable references MUST use the NEXT_PUBLIC_ prefix and contain no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) in variable names.
- **R-PUBCONF-003** MUST: Server-side only environment variables (without NEXT_PUBLIC_ prefix) MUST NOT be referenced in client-side component files.
- **R-PUBCONF-004** MUST: Build process MUST complete successfully with all required NEXT_PUBLIC_ variables defined.
- **R-PUBCONF-005** SHOULD: Optional integrations SHOULD use conditional rendering pattern (process.env.NEXT_PUBLIC_VAR && <Component />) to prevent runtime errors when configuration is missing.
- **R-PUBCONF-006** SHOULD: Public environment variables SHOULD be documented in .env.example and .env.local.example files with descriptions and example values.

### Verify

```bash
# Check for NEXT_PUBLIC_ usage in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'

# Count non-public environment variable references in client code
grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l

# Verify build completes successfully
npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'No public env vars in build output'
```

**Accept when:**
- All client-side environment variable references use NEXT_PUBLIC_ prefix and no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in NEXT_PUBLIC_ variable names
- Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files
- Build process completes successfully with all required NEXT_PUBLIC_ variables defined
- Public environment variables are documented in environment templates with descriptions and example values
- Optional integrations use conditional rendering with fallbacks for missing configuration

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side configuration in Next.js applications. Violations MUST be caught during code review and CI pipeline checks before merge.
</enforcement>