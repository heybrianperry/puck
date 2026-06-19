# Isolate Environment Variable Access to process.env for Secrets and Configuration: Secrets Accessed Process

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SECRETS-001** MUST NOT: Secrets accessed from process.env MUST NOT be logged, exposed in error messages, or included in client-side bundles.

### Verify

```bash
# Identify all process.env accesses
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for specific environment variables
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify environment variable documentation exists
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'

# Scan for hardcoded secrets or credentials
grep -r 'password\|secret\|api[_-]?key\|token' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules | grep -v '^\.env'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables
- Secrets are not logged, exposed in error messages, or included in client-side bundles
- .env.example or equivalent documentation exists describing all required environment variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All process.env accesses must be reviewed to ensure secrets are not logged, exposed in error messages, or bundled for client-side use.
</enforcement>