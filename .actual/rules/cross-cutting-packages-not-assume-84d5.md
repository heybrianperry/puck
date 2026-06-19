# Use process.env for Runtime Configuration in CLI and Component Packages: Packages Not Assume

These rules are ALWAYS ACTIVE for all CLI tools in packages/create-puck-app, React components in packages/core/components, and any package requiring runtime environment detection.

### Rules

- **R-PROCENV-001** MUST_NOT: Packages MUST NOT assume environment variables are always present; provide fallback behavior for missing values.
- **R-PROCENV-002** MUST: All CLI tools in packages/create-puck-app MUST access process.env.npm_config_user_agent for package manager detection with fallback values.
- **R-PROCENV-003** MUST: All React components in packages/core MUST check process.env.NODE_ENV for environment-specific behavior with fallback values.
- **R-PROCENV-004** MUST: No direct process.env access MUST occur without documented fallback values or validation.
- **R-PROCENV-005** MUST: All environment variables used by each package MUST be documented in README.md, including expected values and default behavior.
- **R-PROCENV-006** SHOULD: Use logical OR (||) or nullish coalescing (??) operators to provide fallback behavior when environment variables are undefined.
- **R-PROCENV-007** SHOULD: For browser-targeted packages, ensure bundler configuration includes process.env replacement and test that process.env references are eliminated from production bundles.

### Verify

```bash
# Check for process.env usage patterns in CLI and component packages
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -E '(npm_config_user_agent|NODE_ENV)'

# Verify process.env exists in key files
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js', 'packages/core/components/Puck/index.tsx']; files.forEach(f => { if (fs.existsSync(f)) { const content = fs.readFileSync(f, 'utf8'); if (!content.includes('process.env')) { console.error('Missing process.env in ' + f); process.exit(1); } } });"

# Check for undocumented process.env usage (excluding standard public vars)
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v -E '(npm_config_user_agent|NODE_ENV|NEXT_PUBLIC_|VITE_)' || true
```

**Accept when:**
- All CLI tools in packages/create-puck-app access process.env.npm_config_user_agent for package manager detection
- All React components in packages/core check process.env.NODE_ENV for environment-specific behavior
- No direct process.env access occurs without fallback values or validation
- Documentation exists for all environment variables used by each package
- All environment variable reads use fallback operators (|| or ??)
- Bundler configuration for browser targets includes process.env replacement

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if process.env access lacks documented fallback behavior. Code review MUST block merge if new environment variables are introduced without documentation. Runtime warnings MUST be logged in development mode when environment variables are missing.
</enforcement>