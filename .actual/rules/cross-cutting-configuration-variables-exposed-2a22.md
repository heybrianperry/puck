# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Configuration Variables Exposed

These rules are ALWAYS ACTIVE for all client-side React components, Next.js pages, and UI components that require deployment-specific configuration or coordinate with internal API endpoints.

### Rules

- **R-CONFIG-001** MUST: Configuration variables exposed to the browser runtime MUST use the `NEXT_PUBLIC_` naming convention to ensure proper inlining during Next.js build process.
- **R-CONFIG-002** MUST: All client-side components that call internal APIs MUST source base URLs and environment flags from `process.env.NEXT_PUBLIC_*` variables rather than hardcoding them.
- **R-CONFIG-003** MUST: No hardcoded environment-specific URLs or flags SHALL exist in component code (excluding test files and documented exceptions per EXC-001).
- **R-CONFIG-004** SHOULD: Create a centralized constants file (e.g., `@/core/lib/config.ts`) that reads `process.env.NEXT_PUBLIC_*` values and exports typed constants.
- **R-CONFIG-005** SHOULD: Add TypeScript type definitions for expected `NEXT_PUBLIC_` variables in `next-env.d.ts` or a custom types file to enable IDE autocomplete and type checking.
- **R-CONFIG-006** SHOULD: Include runtime validation at application bootstrap that checks for required `NEXT_PUBLIC_` variables and logs clear error messages if missing.
- **R-CONFIG-007** MUST: All `NEXT_PUBLIC_` variables MUST be documented in `.env.example` with descriptions and example values per environment.
- **R-CONFIG-008** MUST NOT: Secret values such as API keys or tokens SHALL NEVER use the `NEXT_PUBLIC_` prefix.

### Verify

```bash
# Check for non-standard NEXT_PUBLIC_ usage patterns
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'

# Check for hardcoded URLs in component code
grep -r 'const.*=.*['"']http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'

# Verify configuration is documented
test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'
```

**Accept when:**
- All client-side components that call internal APIs source base URLs and environment flags from `process.env.NEXT_PUBLIC_*` variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All `NEXT_PUBLIC_` variables are documented in `.env.example` with descriptions and example values
- Runtime validation is present at application bootstrap for required configuration values
- TypeScript type definitions exist for expected `NEXT_PUBLIC_` variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for client-side configuration. Violations detected by automated CI checks or code review MUST block merge. Exceptions require explicit documentation referencing EXC-001 and team lead approval.
</enforcement>