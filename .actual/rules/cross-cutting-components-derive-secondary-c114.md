# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Components Derive Secondary

These rules are ALWAYS ACTIVE for all client-side React components and Next.js pages that require access to deployment-specific configuration values at browser runtime, particularly those coordinating with internal API endpoints.

### Rules

- **R-NEXTPUB-001** MAY: Components MAY derive secondary configuration values from primary process.env sources when logical transformations are required.
- **R-NEXTPUB-002** MUST: All client-side components that call internal APIs source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables.
- **R-NEXTPUB-003** MUST: No hardcoded environment-specific URLs or flags shall exist in component code (excluding test files and documented exceptions).
- **R-NEXTPUB-004** MUST: All NEXT_PUBLIC_ variables shall be documented in .env.example with descriptions and example values.
- **R-NEXTPUB-005** SHOULD: Create a centralized constants file (e.g., @/core/lib/config.ts) that reads process.env.NEXT_PUBLIC_* values and exports typed constants.
- **R-NEXTPUB-006** SHOULD: Add TypeScript type definitions for expected NEXT_PUBLIC_ variables in next-env.d.ts or a custom types file.
- **R-NEXTPUB-007** SHOULD: Include runtime validation at application bootstrap that checks for required NEXT_PUBLIC_ variables and logs clear error messages if missing.

### Verify

```bash
# Check for non-standard NEXT_PUBLIC_ usage beyond approved set
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'

# Check for hardcoded URLs not sourced from process.env
grep -r 'const.*=.*['"']http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'

# Verify configuration is documented
test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'
```

**Accept when:**
- All client-side components that call internal APIs source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions and example values

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to client-side components that use environment configuration.
</enforcement>