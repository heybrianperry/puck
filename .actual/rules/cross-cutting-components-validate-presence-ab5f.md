# Isolate Environment Variable Access to process.env for Secrets and Configuration: Components Validate Presence

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ENV-001** SHOULD: Components SHOULD validate the presence and format of required environment variables and provide clear error messages when missing.

### Verify

```bash
# Identify all process.env accesses in the codebase
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for specific environment variables referenced in the ADR
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify environment variable documentation exists
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables

<enforcement>
Clause Code MUST NOT skip or defer verification. All process.env accesses must be validated at component initialization or application startup, and required environment variables must be documented with expected formats and example values.
</enforcement>