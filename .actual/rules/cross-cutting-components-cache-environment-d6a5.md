# Isolate Environment Variable Access to process.env for Secrets and Configuration: Components Cache Environment

These rules are ALWAYS ACTIVE for all files accessing runtime configuration and secrets through process.env, specifically in CLI tooling (create-puck-app) and core runtime components (Puck component).

### Rules

- **R-ENV-001** MAY: Components MAY cache environment variable values after initial access to avoid repeated process.env lookups.
- **R-ENV-002** MUST: Access process.env variables at component initialization or application startup rather than during module evaluation to support testing and mocking.
- **R-ENV-003** MUST: Document all required environment variables in README or deployment documentation with expected formats and example values.
- **R-ENV-004** MUST: Implement validation logic that checks for required environment variables and fails fast with clear error messages if missing.
- **R-ENV-005** SHOULD: Consider using dotenv or similar tools for local development to load environment variables from .env files.
- **R-ENV-006** MUST NOT: Hardcode secrets or credentials in source files; all sensitive values must be injected via process.env.
- **R-ENV-007** SHOULD: Use namespaced prefixes for application-specific environment variables to avoid naming conflicts with third-party dependencies.

### Verify

```bash
# Identify all process.env accesses
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for specific configuration variables
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify environment variable documentation exists
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'

# Scan for hardcoded secrets or credentials
grep -r 'password\|secret\|api[_-]?key\|token' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules | grep -v '\bprocess\.env\b'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables
- A .env.example file exists documenting all required environment variables
- All environment variable accesses occur at component initialization or application startup, not during module evaluation

<enforcement>
Clause Code MUST NOT skip or defer verification. All process.env accesses must be reviewed for compliance with caching, validation, and documentation requirements before merge.
</enforcement>