# Use process.env for Runtime Configuration in CLI and Component Packages: Packages Use Process

These rules are ALWAYS ACTIVE for all files in CLI tools (packages/create-puck-app), React components (packages/core/components), and any package requiring runtime environment detection.

### Rules

- **R-PROC-001** SHOULD: Packages SHOULD use process.env as the primary runtime configuration source rather than configuration files.
- **R-PROC-002** MUST: All CLI tools MUST use process.env.npm_config_user_agent to detect the invoking package manager (npm, yarn, pnpm).
- **R-PROC-003** MUST: All React components MUST check process.env.NODE_ENV === 'development' to conditionally enable development-only warnings and debugging features.
- **R-PROC-004** MUST: All process.env reads MUST provide fallback values using logical OR (||) or nullish coalescing (??) operators.
- **R-PROC-005** MUST: All environment variables used by each package MUST be documented in README.md, including expected values and default behavior.
- **R-PROC-006** MUST: For browser-targeted packages, bundler configuration MUST include process.env replacement; production bundles MUST NOT contain process.env references.
- **R-PROC-007** MUST: No direct process.env access is permitted without documented fallback behavior or validation.
- **R-PROC-008** SHOULD: Environment variable values SHOULD NOT be logged or exposed in error messages to prevent leaking sensitive information.

### Verify

```bash
# Check for process.env usage patterns in CLI and component packages
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -E '(npm_config_user_agent|NODE_ENV)'

# Verify critical files contain process.env references
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js', 'packages/core/components/Puck/index.tsx']; files.forEach(f => { if (fs.existsSync(f)) { const content = fs.readFileSync(f, 'utf8'); if (!content.includes('process.env')) { console.error('Missing process.env in ' + f); process.exit(1); } } });"

# Check for undocumented process.env usage (excluding standard bundler variables)
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v -E '(npm_config_user_agent|NODE_ENV|NEXT_PUBLIC_|VITE_)' || true
```

**Accept when:**
- All CLI tools in packages/create-puck-app access process.env.npm_config_user_agent for package manager detection
- All React components in packages/core check process.env.NODE_ENV for environment-specific behavior
- No direct process.env access occurs without fallback values or validation
- Documentation exists for all environment variables used by each package
- Browser-targeted packages have bundler configuration verified to replace process.env at build time
- All environment variable reads include fallback behavior using || or ?? operators

<enforcement>
Claude Code MUST NOT skip or defer verification. All process.env usage patterns MUST be verified before accepting code changes. CI pipeline MUST fail if process.env access lacks documented fallback behavior. Code review MUST block merge if new environment variables are introduced without documentation.
</enforcement>