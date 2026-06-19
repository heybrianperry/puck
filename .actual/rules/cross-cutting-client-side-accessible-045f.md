# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Client Side Accessible

These rules are ALWAYS ACTIVE for all Next.js client-side components and files that require runtime configuration for external service integration, API routing, and feature flags.

### Rules

- **R-NEXTPUB-001** MUST: Client-side accessible environment variables MUST use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-002** MUST: Sensitive credentials (API_KEY, SECRET, PASSWORD, TOKEN) MUST NOT be prefixed with NEXT_PUBLIC_.
- **R-NEXTPUB-003** SHOULD: Use conditional rendering pattern (process.env.NEXT_PUBLIC_VAR && <Component />) for optional integrations to prevent runtime errors when configuration is missing.
- **R-NEXTPUB-004** SHOULD: Document all NEXT_PUBLIC_ variables in .env.example and .env.local.example files with descriptions and example values.
- **R-NEXTPUB-005** MAY: Use relative URLs where possible in NEXT_PUBLIC_ variables to avoid leaking internal infrastructure details.

### Verify

```bash
# Check for NEXT_PUBLIC_ usage in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'

# Count non-public environment variable references in client-side code
grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l

# Verify build completes successfully with public env vars
npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'No public env vars in build output'
```

**Accept when:**
- All client-side environment variable references use NEXT_PUBLIC_ prefix and no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in NEXT_PUBLIC_ variable names.
- Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files.
- Build process completes successfully with all required NEXT_PUBLIC_ variables defined.
- All NEXT_PUBLIC_ variables are documented in environment template files with example values.

<enforcement>
Claude Code MUST NOT skip or defer verification. All client-side environment variable usage must be scanned for proper NEXT_PUBLIC_ prefixing and absence of sensitive credential patterns before accepting changes.
</enforcement>