# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Environment Variable Access

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, including React components, build-time configuration files, custom Document and App components, and API routes.

### Rules

- **R-ENV-001** SHOULD: Environment variable access in client components SHOULD be limited to NEXT_PUBLIC_ prefixed variables.
- **R-ENV-002** MUST: Server-side only environment variables (without NEXT_PUBLIC_ prefix) MUST only be accessed in API routes, getServerSideProps, getStaticProps, and middleware.
- **R-ENV-003** MUST: Build-time environment variables MUST only be accessed in next.config.js/mjs for conditional build configuration.
- **R-ENV-004** SHOULD: Direct property access on process.env SHOULD be used (rather than destructuring or dynamic access) to enable Next.js static analysis and tree-shaking optimizations.
- **R-ENV-005** SHOULD: Components SHOULD handle undefined environment variables gracefully without runtime errors.

### Verify

```bash
# Verify NEXT_PUBLIC_ prefixed variables in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Check for non-NEXT_PUBLIC_ environment variable access in client components
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'

# Verify environment variables are documented
test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'
```

**Accept when:**
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors
- All environment variables are documented in .env.example with descriptions and required/optional status

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline checks.
</enforcement>