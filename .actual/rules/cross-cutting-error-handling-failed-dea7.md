# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Error Handling Failed

These rules are ALWAYS ACTIVE for all client-side React components and Next.js pages that fetch data from internal APIs or require deployment-specific configuration.

### Rules

- **R-CONFIG-001** SHOULD: Error handling for failed API requests SHOULD log configuration context (e.g., BASE_URL) to aid debugging of environment-specific issues.

### Verify

```bash
# Check for non-standard NEXT_PUBLIC_ usage
grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'

# Check for hardcoded URLs
grep -r 'const.*=.*['"']http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'

# Verify configuration documentation
test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'
```

**Accept when:**
- All client-side components that call internal APIs source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions and example values
- Error handling in API fetch calls includes configuration context logging for debugging environment-specific failures

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>