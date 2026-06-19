# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Sensitive Credentials Keys

These rules are ALWAYS ACTIVE for all Next.js client-side components, configuration files, and environment variable declarations across the project.

### Rules

- **R-NEXTPUB-001** MUST NOT: Sensitive credentials, API keys, or secrets MUST NOT use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-002** MUST: All client-side environment variable references in React components MUST use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-003** MUST: Server-side only environment variables (without NEXT_PUBLIC_ prefix) MUST NOT be referenced in client-side component files.
- **R-NEXTPUB-004** SHOULD: Use conditional rendering pattern (process.env.NEXT_PUBLIC_VAR && <Component />) for optional integrations to prevent runtime errors when optional configuration is missing.
- **R-NEXTPUB-005** SHOULD: Document all NEXT_PUBLIC_ variables in .env.example and .env.local.example files with descriptions and example values.
- **R-NEXTPUB-006** MAY: Use relative URLs where possible in NEXT_PUBLIC_ variables to avoid leaking internal infrastructure details.

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
- All NEXT_PUBLIC_ variables are documented in environment template files with example values

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated grep-based scanning in CI pipeline MUST check for NEXT_PUBLIC_ usage patterns. Code review MUST verify no sensitive credentials use NEXT_PUBLIC_ prefix. Build-time validation MUST fail if required public environment variables are undefined or if sensitive patterns are detected in NEXT_PUBLIC_ variable names.
</enforcement>