# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Sensitive Credentials Keys

These rules are ALWAYS ACTIVE for all client-side React components, browser-executed code, and Next.js applications requiring runtime configuration for external service integration and feature flags.

### Rules

- **R-NEXTPUB-001** MUST NOT: Sensitive credentials, API keys, or server-only secrets MUST NOT use the NEXT_PUBLIC_ prefix.
- **R-NEXTPUB-002** MUST: All client-side environment variable references in React components and browser-executed code MUST use the NEXT_PUBLIC_ prefix pattern.
- **R-NEXTPUB-003** MUST: Client-side code MUST include conditional checks before accessing NEXT_PUBLIC_ variables to gracefully handle undefined values.
- **R-NEXTPUB-004** SHOULD: Define all NEXT_PUBLIC_ variables in .env.local for local development and document them in .env.example with descriptions.
- **R-NEXTPUB-005** SHOULD: Use TypeScript module augmentation to declare process.env types for NEXT_PUBLIC_ variables, enabling IDE autocomplete and type checking.
- **R-NEXTPUB-006** SHOULD: For API endpoints constructed from NEXT_PUBLIC_BASE_URL, provide fallback to relative paths or document the variable as required for production builds.

### Verify

```bash
# Check for client-side environment variable references without NEXT_PUBLIC_ prefix
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
- TypeScript declarations for NEXT_PUBLIC_ variables are present and properly typed
- .env.example documents all required NEXT_PUBLIC_ variables with descriptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side configuration in Next.js applications. Violations blocking deployment include: (1) client-side process.env access without NEXT_PUBLIC_ prefix, (2) secrets detected in NEXT_PUBLIC_ variables, (3) build failures from missing NEXT_PUBLIC_ variables, and (4) code review findings requiring resolution before merge.
</enforcement>