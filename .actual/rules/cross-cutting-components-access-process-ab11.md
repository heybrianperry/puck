# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Components Access Process

These rules are ALWAYS ACTIVE for all client-side React components and Next.js pages that access deployment-specific configuration or call internal API endpoints.

### Rules

- **R-CONFIG-001** SHOULD: Components SHOULD access process.env configuration values at the module or component initialization scope rather than within render loops.
- **R-CONFIG-002** MUST: All NEXT_PUBLIC_ prefixed variables MUST be documented in .env.example with descriptions and example values per environment.
- **R-CONFIG-003** MUST: Client-side components that call internal APIs MUST source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables, never from hardcoded strings.
- **R-CONFIG-004** MUST: No hardcoded environment-specific URLs or flags SHALL exist in component code (excluding test files and documented exceptions per EXC-001).
- **R-CONFIG-005** SHOULD: A centralized constants file (e.g., @/core/lib/config.ts) SHOULD be created to read process.env.NEXT_PUBLIC_* values and export typed constants.
- **R-CONFIG-006** SHOULD: TypeScript type definitions for expected NEXT_PUBLIC_ variables SHOULD be added in next-env.d.ts or a custom types file to enable IDE autocomplete and type checking.
- **R-CONFIG-007** SHOULD: Runtime validation at application bootstrap SHOULD check for required NEXT_PUBLIC_ variables and log clear error messages if missing.
- **R-CONFIG-008** MUST: Pre-commit hooks and CI checks MUST scan for common secret patterns in NEXT_PUBLIC_ variables to prevent accidental exposure of sensitive values.

### Verify

```bash
# Check for non-standard NEXT_PUBLIC_ usage in components
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'

# Check for hardcoded URLs in components
grep -r 'const.*=.*['"']http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'

# Verify .env.example documentation exists
test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'
```

**Accept when:**
- All client-side components that call internal APIs source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions and example values
- Runtime validation fails fast with clear error messages when required NEXT_PUBLIC_ variables are missing
- Pre-commit hooks and CI checks prevent NEXT_PUBLIC_ prefixing of common secret-related variable names

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CONFIG rules MUST be checked during code review and CI pipeline execution. Violations MUST block merge. Exceptions require explicit approval and documentation referencing EXC-001 criteria.
</enforcement>