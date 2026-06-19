# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Public Environment Variables

These rules are ALWAYS ACTIVE for all Next.js client-side components and files requiring runtime configuration in browser contexts.

### Rules

- **R-PUBENV-001** SHOULD: Public environment variables SHOULD be validated for presence before use in conditional rendering or API calls.

### Verify

```bash
# Check for proper NEXT_PUBLIC_ usage in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'

# Verify no sensitive patterns in NEXT_PUBLIC_ variable names
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -E '(API_KEY|SECRET|PASSWORD|TOKEN)' && echo 'FAIL: Sensitive patterns found' || echo 'PASS: No sensitive patterns'

# Check that server-side env vars are not used in client components
grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l

# Verify build completes successfully
npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'No public env vars in build output'
```

**Accept when:**
- All client-side environment variable references use NEXT_PUBLIC_ prefix and no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in NEXT_PUBLIC_ variable names
- Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files
- Build process completes successfully with all required NEXT_PUBLIC_ variables defined
- Public environment variables are checked for presence before use in conditional rendering or API calls

<enforcement>
Claude Code MUST NOT skip or defer verification. All client-side environment variable usage must be scanned for proper NEXT_PUBLIC_ prefixing and absence of sensitive credential patterns before accepting changes.
</enforcement>