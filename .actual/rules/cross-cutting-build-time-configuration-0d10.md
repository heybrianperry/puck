# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Build Time Configuration

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, React components requiring feature flags or deployment environment detection, build-time configuration files (next.config.mjs, next.config.js), and custom Document and App components (_document.tsx, _app.tsx).

### Rules

- **R-NEXTJS-CONFIG-001** SHOULD: Build-time configuration files SHOULD access environment variables at the module top level for static optimization.
- **R-NEXTJS-CONFIG-002** MUST: Client-side environment variable access MUST use NEXT_PUBLIC_ prefix only.
- **R-NEXTJS-CONFIG-003** MUST: Server-side only variables (without NEXT_PUBLIC_ prefix) MUST be accessed only in API routes, getServerSideProps, getStaticProps, middleware, or configuration files.
- **R-NEXTJS-CONFIG-004** SHOULD: Components SHOULD handle undefined environment variables gracefully without runtime errors using conditional rendering patterns.
- **R-NEXTJS-CONFIG-005** SHOULD: Direct property access on process.env (rather than destructuring or dynamic access) SHOULD be used to enable Next.js static analysis and tree-shaking optimizations.

### Verify

```bash
# Verify NEXT_PUBLIC_ prefixed variables in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Check for server-side variables accessed in client components
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'

# Verify environment variables are documented
test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'
```

**Accept when:**
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors
- Environment variables are documented in .env.example with descriptions

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline checks.
</enforcement>