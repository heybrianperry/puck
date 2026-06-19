# Isolate Environment Variable Access to process.env for Secrets and Configuration: Secrets Sensitive Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SECRETS-001** MUST: All secrets and sensitive configuration values MUST be accessed exclusively through process.env rather than hardcoded in source files.

### Verify

```bash
# Identify all process.env accesses
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for specific environment variables
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify environment variable documentation exists
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'

# Scan for hardcoded secrets or credentials
grep -r 'password\|secret\|api[_-]?key\|token' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules | grep -v '//.*' | grep -v 'process\.env'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables
- .env.example or equivalent documentation exists describing all required environment variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All secrets and sensitive configuration must be accessed through process.env exclusively. Hardcoded credentials or secrets are a critical security violation and must be rejected.
</enforcement>