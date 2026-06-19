# Use process.env for Runtime Configuration in CLI and Component Packages: Environment Variable Access

These rules are ALWAYS ACTIVE for all CLI tools in packages/create-puck-app, React components in packages/core/components, and any package requiring runtime environment detection.

### Rules

- **R-ENV-001** SHOULD: Environment variable access SHOULD occur at runtime rather than being cached at module initialization.
- **R-ENV-002** MUST: All environment variable reads MUST provide fallback values using logical OR (||) or nullish coalescing (??) operators.
- **R-ENV-003** MUST: CLI tools MUST use process.env.npm_config_user_agent to detect package manager (npm, yarn, pnpm).
- **R-ENV-004** MUST: React components MUST check process.env.NODE_ENV === 'development' to conditionally enable development-only warnings and debugging features.
- **R-ENV-005** MUST: All environment variables used by each package MUST be documented in README.md, including expected values and default behavior.
- **R-ENV-006** MUST: Browser-targeted packages MUST ensure bundler configuration includes process.env replacement and verify process.env references are eliminated from production bundles.
- **R-ENV-007** MUST: Environment variable values MUST NOT be logged or exposed in error messages without sanitization to prevent leaking sensitive information.

### Verify

```bash
# Check for process.env usage patterns in CLI and component packages
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -E '(npm_config_user_agent|NODE_ENV)'

# Verify process.env exists in key files
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js', 'packages/core/components/Puck/index.tsx']; files.forEach(f => { if (fs.existsSync(f)) { const content = fs.readFileSync(f, 'utf8'); if (!content.includes('process.env')) { console.error('Missing process.env in ' + f); process.exit(1); } } });"

# Check for undocumented process.env access (excluding known public vars)
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v -E '(npm_config_user_agent|NODE_ENV|NEXT_PUBLIC_|VITE_)' || true
```

**Accept when:**
- All CLI tools in packages/create-puck-app access process.env.npm_config_user_agent for package manager detection
- All React components in packages/core check process.env.NODE_ENV for environment-specific behavior
- No direct process.env access occurs without fallback values or validation
- Documentation exists for all environment variables used by each package
- Bundler configuration correctly replaces process.env references for browser targets
- Environment variable values are not logged or exposed in error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All process.env access patterns must be validated against these rules before code is approved. CI pipeline must fail if process.env access lacks documented fallback behavior.
</enforcement>