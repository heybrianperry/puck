# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Next Public Variables

These rules are ALWAYS ACTIVE for all client-side React components, browser-executed code, and Next.js applications requiring runtime configuration for external service integration and feature flags.

### Rules

- **R-NEXT-001** SHOULD: NEXT_PUBLIC_ variables SHOULD be documented in project configuration files or README with their purpose and expected values.
- **R-NEXT-002** MUST: All client-side environment variable references MUST use the NEXT_PUBLIC_ prefix pattern.
- **R-NEXT-003** MUST: Server-only secrets and credentials MUST NOT be prefixed with NEXT_PUBLIC_.
- **R-NEXT-004** SHOULD: Client-side code SHOULD include conditional checks before accessing NEXT_PUBLIC_ variables to gracefully handle undefined values.
- **R-NEXT-005** SHOULD: NEXT_PUBLIC_ variables SHOULD be defined in .env.local for local development and documented in .env.example with descriptions.
- **R-NEXT-006** SHOULD: TypeScript module augmentation SHOULD be used to declare process.env types for NEXT_PUBLIC_ variables, enabling IDE autocomplete and type checking.

### Verify

```bash
# Check for client-side environment variable usage without NEXT_PUBLIC_ prefix
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v node_modules

# Verify all NEXT_PUBLIC_ references are present in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_'

# Check build process for environment variable warnings
npm run build 2>&1 | grep -i 'environment variable'
```

**Accept when:**
- All client-side environment variable references use the NEXT_PUBLIC_ prefix pattern
- No server-only secrets or credentials are prefixed with NEXT_PUBLIC_
- Build process completes without warnings about missing or misconfigured environment variables
- Client-side code includes conditional checks before accessing NEXT_PUBLIC_ variables
- NEXT_PUBLIC_ variables are documented in project configuration files or README

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests with client-side process.env access without NEXT_PUBLIC_ prefix are blocked pending correction. Security scans detecting secrets in NEXT_PUBLIC_ variables trigger immediate incident response. Build failures from missing NEXT_PUBLIC_ variables block deployment until configuration is provided.
</enforcement>