# Use process.env for Runtime Configuration in CLI and Component Packages: Cli Tools Parse

These rules are ALWAYS ACTIVE for CLI tools in packages/create-puck-app, React components in packages/core/components, and any package requiring runtime environment detection.

### Rules

- **R-CLI-001** MAY: CLI tools MAY parse structured data from environment variables (e.g., user agent strings) to derive configuration.
- **R-CLI-002** MUST: Always provide fallback behavior when environment variables are undefined; use logical OR (||) or nullish coalescing (??) operators.
- **R-CLI-003** MUST: Document all environment variables used by each package in README.md, including expected values and default behavior.
- **R-CLI-004** SHOULD: For CLI tools, use process.env.npm_config_user_agent to detect package manager; parse the user agent string to identify npm, yarn, or pnpm.
- **R-CLI-005** SHOULD: For React components, check process.env.NODE_ENV === 'development' to enable development-only warnings and debugging features.
- **R-CLI-006** MUST: For browser-targeted packages, ensure bundler configuration includes process.env replacement; test that process.env references are eliminated from production bundles.

### Verify

```bash
# Check for process.env usage with npm_config_user_agent or NODE_ENV
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -E '(npm_config_user_agent|NODE_ENV)'

# Verify CLI and component files contain process.env
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js', 'packages/core/components/Puck/index.tsx']; files.forEach(f => { if (fs.existsSync(f)) { const content = fs.readFileSync(f, 'utf8'); if (!content.includes('process.env')) { console.error('Missing process.env in ' + f); process.exit(1); } } });"

# Check for undocumented process.env usage (excluding standard vars)
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v -E '(npm_config_user_agent|NODE_ENV|NEXT_PUBLIC_|VITE_)' || true
```

**Accept when:**
- All CLI tools in packages/create-puck-app access process.env.npm_config_user_agent for package manager detection
- All React components in packages/core check process.env.NODE_ENV for environment-specific behavior
- No direct process.env access occurs without fallback values or validation
- Documentation exists for all environment variables used by each package
- All environment variable reads include fallback values using || or ?? operators
- Browser-targeted packages have verified bundler configuration for process.env replacement

<enforcement>
Claude Code MUST NOT skip or defer verification. All process.env usage must include documented fallback behavior and pass CI pipeline grep-based verification checks before merge.
</enforcement>