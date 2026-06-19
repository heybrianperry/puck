# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Public Configuration Values

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, including React components, build-time configuration files, and custom Document and App components.

### Rules

- **R-PUBCONF-001** MUST: Public configuration values exposed to the browser MUST use the NEXT_PUBLIC_ prefix convention.

### Verify

```bash
# Verify NEXT_PUBLIC_ prefixed variables are accessed via direct process.env property access
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Check for server-side only variables accessed in client components (should be empty)
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'

# Verify environment variables are documented
test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'
```

**Accept when:**
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors
- All environment variables are documented in .env.example with descriptions

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes that introduce or modify environment variable access patterns.
</enforcement>