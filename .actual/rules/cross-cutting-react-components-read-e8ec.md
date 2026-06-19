# Use process.env for Runtime Configuration in CLI and Component Packages: React Components Read

These rules are ALWAYS ACTIVE for all CLI tools in packages/create-puck-app, React components in packages/core/components, and any package requiring runtime environment detection.

### Rules

- **R-PROCENV-001** MUST: React components MUST read process.env.NODE_ENV to determine development vs production mode.
- **R-PROCENV-002** MUST: All environment variable reads MUST provide fallback values using logical OR (||) or nullish coalescing (??) operators.
- **R-PROCENV-003** MUST: CLI tools MUST use process.env.npm_config_user_agent to detect package manager (npm, yarn, pnpm).
- **R-PROCENV-004** SHOULD: All environment variables used by each package SHOULD be documented in README.md, including expected values and default behavior.
- **R-PROCENV-005** SHOULD: Browser-targeted packages SHOULD ensure bundler configuration includes process.env replacement and verify process.env references are eliminated from production bundles.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All process.env reads must include fallback values and be documented. CI pipeline must fail if these rules are violated.
</enforcement>