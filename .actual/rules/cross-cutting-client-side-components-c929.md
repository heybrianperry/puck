# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Client Side Components

These rules are ALWAYS ACTIVE for all client-side React components and browser-executed code in Next.js applications that require runtime configuration values for external service integration and feature flags.

### Rules

- **R-NEXT-001** MUST: Client-side components MUST access environment variables through `process.env.NEXT_PUBLIC_*` syntax.

### Verify

```bash
# Check for NEXT_PUBLIC_ prefixed environment variable usage in client-side code
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_'

# Scan for non-prefixed process.env access in client-side code (potential violations)
grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v node_modules

# Verify build completes without environment variable warnings
npm run build 2>&1 | grep -i 'environment variable'
```

**Accept when:**
- All client-side environment variable references use the `NEXT_PUBLIC_` prefix pattern
- No server-only secrets or credentials are prefixed with `NEXT_PUBLIC_`
- Build process completes without warnings about missing or misconfigured environment variables
- Client-side code includes conditional checks before accessing `NEXT_PUBLIC_` variables
- Code review verification confirms `NEXT_PUBLIC_` prefix usage in all client-side `process.env` access

<enforcement>
Claude Code MUST NOT skip or defer verification of NEXT_PUBLIC_ prefix compliance in client-side components. All client-side environment variable access must be validated against these rules before approval.
</enforcement>