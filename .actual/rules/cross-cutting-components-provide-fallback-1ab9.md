# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Components Provide Fallback

These rules are ALWAYS ACTIVE for all Next.js client-side components and browser-accessible code requiring runtime configuration in this project.

### Rules

- **R-NEXTPUB-001** MAY: Components MAY provide fallback behavior when optional public environment variables are undefined.
- **R-NEXTPUB-002** MUST: All client-side environment variable references use the NEXT_PUBLIC_ prefix and contain no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) in variable names.
- **R-NEXTPUB-003** MUST: Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files.
- **R-NEXTPUB-004** MUST: Build process completes successfully with all required NEXT_PUBLIC_ variables defined.

### Verify

```bash
# Check for NEXT_PUBLIC_ usage patterns and exclude sensitive keywords
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'

# Count non-public environment variable references in client code (should be minimal/zero)
grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l

# Verify build completes without NEXT_PUBLIC_ configuration errors
npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'No public env vars in build output'
```

**Accept when:**
- All client-side environment variable references use NEXT_PUBLIC_ prefix and no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in NEXT_PUBLIC_ variable names
- Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files
- Build process completes successfully with all required NEXT_PUBLIC_ variables defined

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Automated grep-based scanning in CI pipeline and build-time validation are mandatory before accepting changes.
</enforcement>