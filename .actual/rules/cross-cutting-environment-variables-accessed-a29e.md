# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Environment Variables Accessed

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, including React components, build-time configuration files, and custom Document and App components.

### Rules

- **R-ENV-001** MUST: Environment variables MUST be accessed via process.env with explicit property access (e.g., process.env.NEXT_PUBLIC_BASE_URL).

### Verify

```bash
# Verify NEXT_PUBLIC_ variables use direct process.env property access
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Check for server-side variables in client components (should only appear in API routes, getServerSideProps, middleware, or config)
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