# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Components That Conditionally

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, React components requiring feature flags or deployment environment detection, build-time configuration files (next.config.mjs, next.config.js), and custom Document and App components (_document.tsx, _app.tsx).

### Rules

- **R-ENV-001** MUST: Components that conditionally render based on environment variables MUST handle undefined values gracefully.

### Verify

```bash
# Verify NEXT_PUBLIC_ prefixed environment variables are accessed directly via process.env
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Verify server-side only variables are not accessed in client components
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'

# Verify environment variables are documented
test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'
```

**Accept when:**
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors
- Environment variables are documented in .env.example with descriptions and required/optional status

<enforcement>
Clause R-ENV-001 verification is mandatory. Code review MUST verify NEXT_PUBLIC_ prefix usage for client-side variables. ESLint rules or custom linting MUST detect process.env access patterns. CI pipeline MUST scan for potential secret exposure in NEXT_PUBLIC_ variables. Violations result in CI build failure and code review block.
</enforcement>