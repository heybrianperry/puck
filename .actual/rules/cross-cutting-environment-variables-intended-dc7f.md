# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Environment Variables Intended

These rules are ALWAYS ACTIVE for all client-side React components, browser-executed code, and Next.js applications requiring runtime configuration values for external service integration and feature flags.

### Rules

- **R-ENV-001** MUST: All environment variables intended for client-side access MUST use the NEXT_PUBLIC_ prefix.
- **R-ENV-002** MUST: Server-side API routes, getServerSideProps functions, database connection strings, credentials, and private API keys MUST NOT use the NEXT_PUBLIC_ prefix.
- **R-ENV-003** SHOULD: Implement conditional logic (if checks) before accessing NEXT_PUBLIC_ variables to gracefully handle undefined values.
- **R-ENV-004** SHOULD: Use TypeScript module augmentation to declare process.env types for NEXT_PUBLIC_ variables, enabling IDE autocomplete and type checking.
- **R-ENV-005** SHOULD: Define all NEXT_PUBLIC_ variables in .env.local for local development and document them in .env.example with descriptions.
- **R-ENV-006** MAY: For API endpoints constructed from NEXT_PUBLIC_BASE_URL, provide fallback to relative paths or document the variable as required for production builds.

### Verify

```bash
# Check for client-side environment variable references without NEXT_PUBLIC_ prefix
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v node_modules

# Verify all NEXT_PUBLIC_ variables are used in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_'

# Check build process for environment variable warnings
npm run build 2>&1 | grep -i 'environment variable'
```

**Accept when:**
- All client-side environment variable references use the NEXT_PUBLIC_ prefix pattern
- No server-only secrets or credentials are prefixed with NEXT_PUBLIC_
- Build process completes without warnings about missing or misconfigured environment variables
- Client-side code includes conditional checks before accessing NEXT_PUBLIC_ variables
- TypeScript declarations for NEXT_PUBLIC_ variables are present and properly typed
- .env.example documents all required NEXT_PUBLIC_ variables with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All client-side environment variable access must be validated against the NEXT_PUBLIC_ prefix requirement before code is approved.
</enforcement>