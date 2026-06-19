# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Components That Coordinate

These rules are ALWAYS ACTIVE for all client-side React components and Next.js pages that coordinate with internal APIs and require deployment-specific configuration at browser runtime.

### Rules

- **R-COORD-001** MUST: Components that coordinate with internal APIs MUST NOT hardcode environment-specific values such as base URLs, deployment flags, or version identifiers.
- **R-COORD-002** MUST: All client-side components that call internal APIs MUST source base URLs and environment flags from `process.env.NEXT_PUBLIC_*` variables.
- **R-COORD-003** MUST: All `NEXT_PUBLIC_*` variables MUST be documented in `.env.example` with descriptions and example values per environment.
- **R-COORD-004** SHOULD: Create a centralized constants file (e.g., `@/core/lib/config.ts`) that reads `process.env.NEXT_PUBLIC_*` values and exports typed constants.
- **R-COORD-005** SHOULD: Add TypeScript type definitions for expected `NEXT_PUBLIC_*` variables in `next-env.d.ts` or a custom types file to enable IDE autocomplete and type checking.
- **R-COORD-006** SHOULD: Include runtime validation at application bootstrap that checks for required `NEXT_PUBLIC_*` variables and logs clear error messages if missing.

### Verify

```bash
# Check for non-standard NEXT_PUBLIC_ usage patterns
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'

# Check for hardcoded URLs in components
grep -r 'const.*=.*['"']http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'

# Verify .env.example documentation exists
test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'
```

**Accept when:**
- All client-side components that call internal APIs source base URLs and environment flags from `process.env.NEXT_PUBLIC_*` variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All `NEXT_PUBLIC_*` variables are documented in `.env.example` with descriptions and example values
- Runtime validation is present at application bootstrap for required configuration values

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to components that coordinate with internal APIs. Code review MUST include a checklist item confirming that new components use centralized configuration constants sourced from `process.env.NEXT_PUBLIC_*` variables.
</enforcement>