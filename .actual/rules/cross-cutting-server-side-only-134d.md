# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Server Side Only

These rules are ALWAYS ACTIVE for Next.js applications using environment variables for runtime configuration, including React components, API routes, build-time configuration files, and custom Document and App components.

### Rules

- **R-ENV-001** MUST: Access server-side only variables (without NEXT_PUBLIC_ prefix) exclusively in API routes, getServerSideProps, getStaticProps, middleware, and configuration files (next.config.js/mjs).
- **R-ENV-002** MUST: Use direct property access on process.env (e.g., `process.env.NEXT_PUBLIC_VAR`) rather than destructuring or dynamic access to enable Next.js static analysis and tree-shaking optimizations.
- **R-ENV-003** MUST: Never access server-side only environment variables in client-side React components or browser-exposed code.
- **R-ENV-004** SHOULD: Handle undefined environment variables gracefully in components using conditional rendering patterns like `{process.env.NEXT_PUBLIC_VAR && <Component />}`.
- **R-ENV-005** SHOULD: Create a TypeScript declaration file (e.g., env.d.ts) to type process.env with all expected NEXT_PUBLIC_ variables for IDE autocomplete and type safety.
- **R-ENV-006** SHOULD: Document all environment variables in a .env.example file with descriptions and whether they are required or optional.
- **R-ENV-007** MAY: Use a validation library like zod or joi to validate environment variables at build time in next.config.js.

### Verify

```bash
# Check for NEXT_PUBLIC_ prefixed variables in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'

# Check for server-side only variables accessed outside allowed contexts
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'

# Verify environment variables are documented
test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'
```

**Accept when:**
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, getStaticProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors
- All environment variables are documented in .env.example with descriptions
- No common secret patterns are detected in NEXT_PUBLIC_ variable names

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST check NEXT_PUBLIC_ prefix usage for client-side variables. CI pipeline MUST scan for potential secret exposure in NEXT_PUBLIC_ variables. Build MUST fail if server-side only variables are accessed in client components without NEXT_PUBLIC_ prefix.
</enforcement>