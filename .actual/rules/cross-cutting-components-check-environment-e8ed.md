# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Components Check Environment

These rules are ALWAYS ACTIVE for all client-side React components and browser-executed code in Next.js applications that require runtime configuration values for external service integration, feature flags, analytics, and UI behavior switches.

### Rules

- **R-NEXTPUB-001** SHOULD: Components SHOULD check for environment variable existence before using values (e.g., conditional rendering).

### Verify

```bash
# Check for NEXT_PUBLIC_ prefixed environment variables in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_'

# Scan for non-prefixed environment variables in client-side code (potential violations)
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v node_modules

# Check build process for environment variable warnings
npm run build 2>&1 | grep -i 'environment variable'
```

**Accept when:**
- All client-side environment variable references use the NEXT_PUBLIC_ prefix pattern
- No server-only secrets or credentials are prefixed with NEXT_PUBLIC_
- Build process completes without warnings about missing or misconfigured environment variables
- Client-side code includes conditional checks before accessing NEXT_PUBLIC_ variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All client-side environment variable access must be validated against the NEXT_PUBLIC_ prefix requirement and conditional existence checks before code is approved.
</enforcement>